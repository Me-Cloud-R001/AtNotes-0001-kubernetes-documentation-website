---
title: Handling retriable and non-retriable pod failures with Pod failure policy
content_type: task
min-kubernetes-server-version: v1.25
weight: 60
---

{{< feature-state for_k8s_version="v1.25" state="alpha" >}}

<!-- overview -->

This document shows you how to use the
[Pod failure policy](/docs/concepts/workloads/controllers/job#pod-failure-policy),
in combination with the default
[Pod backoff failure policy](/docs/concepts/workloads/controllers/job#pod-backoff-failure-policy),
to improve the control over the handling of container- or Pod-level failure
within a {{<glossary_tooltip text="Job" term_id="job">}}.

The definition of Pod failure policy may help you to:
* better utilize the computational resources by avoiding unnecessary Pod retries.
* avoid Job failures due to Pod disruptions (such {{<glossary_tooltip text="preemption" term_id="preemption" >}},
{{<glossary_tooltip text="API-initiated eviction" term_id="api-eviction" >}}
or {{<glossary_tooltip text="taint" term_id="taint" >}}-based eviction).

## {{% heading "prerequisites" %}}

You should already be familiar with the basic use of [Job](/docs/concepts/workloads/controllers/job/).

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!-- steps -->

{{< note >}}
As the features are in Alpha, prepare the Kubernetes cluster with the two
[feature gates](/docs/reference/command-line-tools-reference/feature-gates/)
enabled: `JobPodFailurePolicy` and `PodDisruptionsCondition`.
{{< /note >}}

## Using Pod failure policy to avoid unnecessary Pod retries

With the following example, you can learn how to use Pod failure policy to
avoid unnecessary Pod restarts when a Pod failure indicates a non-retriable
software bug.

First, create a Job based on the config:

{{< codenew file="/controllers/job-pod-failure-policy-failjob.yaml" >}}

by running:

```sh
kubectl create -f job-pod-failure-policy-failjob.yaml
```

After around 30s the entire Job should be terminated. Inspect the status of the Job by running:
```sh
kubectl get jobs -l job-name=job-pod-failure-policy-failjob -o yaml
```

In the Job status, see a job `Failed` condition with the field `reason`
equal `PodFailurePolicy`. Additionally, the `message` field contains a
more detailed information about the Job termination, such as:
`Container main for pod default/job-pod-failure-policy-failjob-8ckj8 failed with exit code 42 matching FailJob rule at index 0`.

For comparison, if the Pod failure policy was disabled it would take 6 retries
of the Pod, taking at least 2 minutes.

### Clean up

Delete the Job you created:
```sh
kubectl delete jobs/job-pod-failure-policy-failjob
```
The cluster automatically cleans up the Pods.

## Using Pod failure policy to ignore Pod disruptions

With the following example, you can learn how to use Pod failure policy to
ignore Pod disruptions from incrementing the Pod retry counter towards the
`.spec.backoffLimit` limit.

{{< caution >}}
Timing is important for this example, so you may want to read the steps before
execution. In order to trigger a Pod disruption it is important to drain the
node while the Pod is running on it (within 90s since the Pod is scheduled).
{{< /caution >}}

1. Create a Job based on the config:

{{< codenew file="/controllers/job-pod-failure-policy-ignore.yaml" >}}

by running:

```sh
kubectl create -f job-pod-failure-policy-ignore.yaml
```

2. Run this command to check the `nodeName` the Pod is scheduled to:

```sh
nodeName=$(kubectl get pods -l job-name=job-pod-failure-policy-ignore -o jsonpath='{.items[0].spec.nodeName}')
```

3. Drain the node to evict the Pod before it completes (within 90s):
```sh
kubectl drain nodes/$nodeName --ignore-daemonsets --grace-period=0
```

4. Inspect the `.status.failed` to check the counter for the Job is not incremented:
```sh
kubectl get jobs -l job-name=job-pod-failure-policy-ignore -o yaml
```

5. Uncordon the node:
```sh
kubectl uncordon nodes/$nodeName
```

The Job resumes and succeeds.

For comparison, if the Pod failure policy was disabled the Pod disruption would
result in terminating the entire Job (as the `.spec.backoffLimit` is set to 0).

### Cleaning up

Delete the Job you created:
```sh
kubectl delete jobs/job-pod-failure-policy-ignore
```
The cluster automatically cleans up the Pods.

## Alternatives

You could rely solely on the
[Pod backoff failure policy](/docs/concepts/workloads/controllers/job#pod-backoff-failure-policy),
by specifying the Job's `.spec.backoffLimit` field. However, in many situations
it is problematic to find a balance between setting the a low value for `.spec.backoffLimit`
 to avoid unnecessary Pod retries, yet high enough to make sure the Job would
not be terminated by Pod disruptions.
