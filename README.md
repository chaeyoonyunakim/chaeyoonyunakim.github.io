# Data Science Journal

Chaeyoon Kim — Data Scientist based in London.

Built with Jekyll. Custom layout, no external theme dependencies.

## Structure

- `_layouts/` — `default.html` (site shell + CSS), `post.html` (article + LinkedIn redirect)
- `_posts/` — writing, linked or redirected to LinkedIn Pulse
- `index.html` — homepage (hero, about, writing)
- `_config.yml` — site metadata

## Adding a post

```yaml
---
layout: post
title: "Your title"
date: 2026-01-01
redirect_to: https://www.linkedin.com/pulse/...  # optional: redirects to LinkedIn
---
```
