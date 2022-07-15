---
title: Hugo Cheat Sheet
---
## Cheat-sheet Ace-Documentation Theme

With the site created and the theme installed, it's time to run the server and create content.

Ace is a theme for <a href="https://gohugo.io" target="_blank">Hugo</a>, a fast static website generator written in Go, that allows you to easily write well organized and clean documentation for your projects. It's as easy as writing your content in Markdown, running Hugo to generate static HTML, CSS and Javascript files and deploying those to your web server.

## Source Code embedding

```bash
hugo server
```

```markdown
+++
title = "Usage"
description = ""
weight = 2
+++
```

## Important Bash commands

```bash
hugo new site . --format yaml --configDir config --config config/config.yaml, config/config-dev.yaml, config/config-live.yaml  --force

hugo server
```

---
