# hyperworks.nu

Personal site for **Matthew Campbell** (@kanwisher) — work, blog, talks, and future YouTube.

Formerly **kanwisher.com**. Static site built with [Hugo](https://gohugo.io/) and Markdown.

## Quick start

```bash
# requires Hugo extended (brew install hugo)
hugo server -D
# open http://localhost:1313
```

Build for production:

```bash
hugo
# output in public/  → deploy to S3 / Cloudflare / any static host
```

## Content

| Path | Purpose |
|------|---------|
| `content/blog/*.md` | Blog posts |
| `content/talks/*.md` | Conference / meetup talks |
| `content/about.md` | About |
| `content/work.md` | Work / projects |
| `static/images/` | Images (headshot, etc.) |
| `static/slides/` | Optional PDF slide decks |
| `layouts/` | HTML templates (minimal custom theme) |
| `assets/css/main.css` | Styles |

### Adding a blog post

```bash
hugo new blog/my-post-title.md
# edit content/blog/my-post-title.md
```

### Adding a talk

Create `content/talks/slug.md`:

```yaml
---
title: "Talk title"
date: 2018-05-05
year: 2018
event: GopherCon Singapore
city: Singapore
youtube: k0-WyZCKF5I   # optional YouTube video ID
slides: /slides/foo.pdf # optional
tags: [go, databases]
description: One-line summary.
---

Talk abstract / notes in Markdown…
```

## Related local archives (not in this repo)

- Videos + catalog: `~/Archives/hyperworks-media/`
- Original slide sources: `~/projects/me/conferences/slides/`

## Deploy (Netlify)

This repo includes `netlify.toml`:

| Setting | Value |
|---------|--------|
| Build command | `hugo --gc --minify` |
| Publish dir | `public` |
| Hugo version | set in `netlify.toml` (`HUGO_VERSION`) |

1. Push this repo to GitHub/GitLab.
2. [New site from Git](https://app.netlify.com/start) → pick the repo.
3. Netlify reads `netlify.toml` automatically.
4. Add custom domain **hyperworks.nu** in Domain settings (DNS: Netlify nameservers or a CNAME/ALIAS).

Local production check:

```bash
hugo --gc --minify
npx serve public   # optional smoke test
```

Theme is **dark by default** (`color-scheme: dark` on `html` / meta).
