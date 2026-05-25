# chaeyoonyunakim.github.io

Chaeyoon Kim — Data Scientist based in London.

Built with Jekyll. Custom layout, no external theme dependencies.

## Structure

```
_layouts/
  default.html   # site shell, all CSS, nav, footer
  post.html      # writing page: renders full content or redirects to LinkedIn
_posts/          # all writing entries (Markdown)
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

## Writing: two types of post

The writing section has two sub-groups, controlled by the `type` front matter field.

### Reflections & project notes (`type: reflection`)

Full on-site Markdown content — thoughts, lessons learnt, or write-ups in the style of NHS England’s [our_work](https://github.com/nhsengland/datascience/tree/main/docs/our_work) pages.

```yaml
---
layout: post
type: reflection
title: "What I learnt building the PWR elasticity model"
date: 2026-05-25
summary: "One-line description shown on the homepage card."
tags: [econometrics, NHS, panel-data]
---

## Background

## What I did

## Lessons learnt

## Outputs
```

### LinkedIn articles (`type: linkedin`)

Redirects to an article published on LinkedIn Pulse.

```yaml
---
layout: post
type: linkedin
title: "Your article title"
date: 2026-01-01
redirect_to: https://www.linkedin.com/pulse/...
---
```

Any post without a `type` field will appear in the LinkedIn sub-group by default.

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
thumb: "/assets/img/your-thumbnail.jpg"  # local path or absolute https:// URL
thumb_alt: "Alt text for thumbnail"      # optional
excerpt: "One-paragraph description shown on the card."
order: 6                     # controls display order (lower = first)
---
```

**Notes:**
- Use `period` not `date` — Jekyll tries to parse `date` as a Ruby date object.
- `thumb` accepts a local path or an absolute URL. Pin external images to a commit SHA for stability.
- CI validates every `thumb` value on each PR: external URLs via `curl --location`, local paths against built `_site/`.
