---
title: "Hello hyperworks.nu"
date: 2011-12-01description: Relaunching kanwisher.com as a static Markdown site for talks, blog, and reverse engineering write-ups.
tags: [meta, site]
image: /images/blog/headers/hello.jpg
---

**kanwisher.com** is becoming **[hyperworks.nu](https://hyperworks.nu)**.

Same person (@kanwisher / Matthew Campbell). New focus:

1. **Blog** — long-form technical posts, including reverse engineering hardware projects and write-ups that feed a future YouTube channel
2. **Talks** — archive of conference talks (Go, DevOps, blockchain, meetups)
3. **Work** — TensorFleet and earlier chapters
4. **Speaking** — booking and past venues

## Stack

Simple on purpose:

- **Hugo** — static site generator
- **Markdown** — all content under `content/`
- No CMS, no runtime DB

```bash
hugo server -D   # local preview
hugo             # build to public/
```

## What's next

- Done: [G'AIM'E light gun RE](/blog/gaime-lightgun-reverse-engineering/) and [OSCR cart reader companions](/blog/oscr-cartreader-macos-android/)
- More RE write-ups from Twitter threads
- PDFs of talk slides under `static/slides/`
- YouTube channel hub when that launches
- Hosted on **Netlify** (`netlify.toml` in repo)

If you want the old Go / microservices talks, start at [Talks](/talks/).
