# chaeyoonyunakim.github.io

Chaeyoon Kim — Data Scientist based in London.

Built with Jekyll. Custom layout, no external theme dependencies.

## Structure

```
_layouts/
  default.html   # site shell, all CSS, nav, footer
  post.html      # article page; supports LinkedIn redirect
_posts/          # writing entries (Markdown), linked or redirected to LinkedIn Pulse
assets/
  img/           # static images (e.g. project poster thumbnails)
index.html       # homepage — hero · about · projects · writing
cv.html          # standalone CV page at /cv
_config.yml      # site metadata (title, url, plugins)
```

## Pages

| Route | File | Description |
|-------|------|-------------|
| `/` | `index.html` | Hero, About (with CV link), Projects, Writing |
| `/cv` | `cv.html` | Full CV — experience, education, community |
| `/:year/:month/:day/:title/` | `_posts/*.md` | Individual writing entries |

## Adding a post

```yaml
---
layout: post
title: "Your title"
date: 2026-01-01
redirect_to: https://www.linkedin.com/pulse/...  # optional: redirects to LinkedIn
---
```

## Adding a project card

Edit the `#projects` section in `index.html`. For a featured card with a thumbnail:

1. Add a JPEG thumbnail to `assets/img/`.
2. Copy the `.project-card--featured` block and update the `href`, `src`, and text.
3. Regular (non-featured) cards omit `.project-card--featured` and the `.project-thumb` div.
