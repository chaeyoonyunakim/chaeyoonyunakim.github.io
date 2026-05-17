# chaeyoonyunakim.github.io

Chaeyoon Kim — Data Scientist based in London.

Built with Jekyll. Custom layout, no external theme dependencies.

## Structure

```
_layouts/
  default.html   # site shell, all CSS, nav, footer
  post.html      # article page; supports LinkedIn redirect
_posts/          # writing entries (Markdown), linked or redirected to LinkedIn Pulse
_projects/       # project cards (Markdown front matter only, output: false)
assets/
  img/           # static images (e.g. project poster thumbnails)
index.html       # homepage — hero · about · projects · writing
cv.html          # standalone CV page at /cv
_config.yml      # site metadata (title, url, plugins, collections)
Gemfile          # Jekyll 4.3 + jekyll-feed, used by CI
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

Create a new file in `_projects/` — no changes to `index.html` needed.

```yaml
---
title: "Project title"
period: "May 2026"            # display string shown on the card (use 'period', not 'date')
context: "Hackathon · NHS"   # shown after period with · separator
github: "https://github.com/chaeyoonyunakim/your-repo"
badge: "In Progress"         # optional pill label (e.g. "1st Place", "In Progress")
featured: true               # optional — spans full grid width, shows thumbnail
thumb: "/assets/img/your-thumbnail.jpg"  # optional, requires featured: true
thumb_alt: "Alt text for thumbnail"      # optional
excerpt: "One-paragraph description shown on the card."
order: 6                     # controls display order (lower = first)
---
```

Note: use `period` not `date` — Jekyll tries to parse `date` as a Ruby date object.

For a featured card with a thumbnail, also add the JPEG to `assets/img/`.
