# hyperworks.nu

Personal site for **Matthew Campbell** (@kanwisher) — work, blog, talks, and future YouTube.

Formerly **kanwisher.com**. Static site built with [Hugo](https://gohugo.io/) and Markdown. Hosted on **Cloudflare**.

## Quick start

```bash
# requires Hugo extended (brew install hugo)
hugo server -D
# open http://localhost:1313
```

Build for production:

```bash
hugo --gc --minify
# output in public/
```

Theme is **dark by default** (`color-scheme: dark`).

## Content

| Path | Purpose |
|------|---------|
| `content/blog/*.md` | Blog posts |
| `content/talks/*.md` | Conference / meetup talks |
| `content/about.md` | About |
| `content/work.md` | Work / projects |
| `static/images/` | Images (headshot, etc.) |
| `static/slides/` | Optional PDF slide decks (keep under 25 MiB/file for Workers) |
| `layouts/` | HTML templates |
| `assets/css/main.css` | Styles |
| `wrangler.toml` | Cloudflare Workers static assets config |
| `static/_headers` | Cloudflare Pages/Workers security headers |

### Adding a blog post

```bash
hugo new content/blog/my-post-title.md
```

### Adding a talk

Create `content/talks/slug.md` with front matter: `title`, `date`, `year`, `event`, `city`, optional `youtube`, `slides`, `tags`, `description`.

## Deploy on Cloudflare

### Option A — Cloudflare Pages (recommended permanent)

Same account that already holds **kanwisher.com** DNS.

1. [Cloudflare dashboard](https://dash.cloudflare.com) → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. Select **`mattkanwisher/hyperworks.nu`**
3. Build settings:

   | Field | Value |
   |-------|--------|
   | Framework preset | Hugo |
   | Build command | `hugo --gc --minify` |
   | Build output directory | `public` |
   | Root directory | `/` |
   | Environment variable | `HUGO_VERSION` = `0.164.0` |

4. **Custom domains** → add **`hyperworks.nu`** (and `www` if you want)  
   - If the zone is already on Cloudflare, it will attach DNS for you.
5. Every push to `master` redeploys.

### Option B — Cloudflare Drop / Wrangler (CLI)

[Cloudflare Drop](https://www.cloudflare.com/drop/) is instant static hosting. Temporary previews expire unless **claimed within 60 minutes**.

```bash
# Node 22+
hugo --gc --minify

# Temporary preview (no login) — claim URL printed in the output
npx wrangler@4.102.0 deploy ./public \
  --name hyperworks-nu \
  --temporary \
  --compatibility-date 2026-07-28

# Permanent (after wrangler login to your real Cloudflare account)
npx wrangler login
npx wrangler deploy ./public --name hyperworks-nu
```

**Limits:** temporary Drop accounts enforce ~**5 MiB per file**. Large slide PDFs are fine on full Pages/Workers (~25 MiB). Host big decks externally or compress if Drop rejects them.

### Git remotes (local)

| Remote | Repo |
|--------|------|
| `hyperworks` | [mattkanwisher/hyperworks.nu](https://github.com/mattkanwisher/hyperworks.nu) — **primary** |
| `origin` | `mattkanwisher/kanwisher_com` — legacy |

```bash
git push hyperworks master
```

## Redirecting kanwisher.com → hyperworks.nu

`kanwisher.com` is already on **Cloudflare DNS**; origin is old **S3** (~2017). You do **not** need GitHub Pages.

After `hyperworks.nu` is live on Cloudflare Pages:

1. Zone **kanwisher.com** → **Rules** → **Redirect Rules** → Create  
2. If: hostname is `kanwisher.com` **or** `www.kanwisher.com`  
3. Then: **301** to  
   `concat("https://hyperworks.nu", http.request.uri.path)`  
   (preserve query string)  
4. Optional later: empty the S3 bucket or set S3 “redirect all requests” as a belt-and-suspenders.

Same Cloudflare account can hold both zones (`kanwisher.com` + `hyperworks.nu`).

## Related local archives (not in this repo)

- Videos + catalog: `~/Archives/hyperworks-media/`
- Original slide sources: `~/projects/me/conferences/slides/`
