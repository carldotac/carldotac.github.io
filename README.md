# carl.ac

Personal site for Carl Lieberman, built with [Hugo](https://gohugo.io/) and the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

The theme is pulled in as a **Hugo Module** (see `go.mod`) — it is *not* forked
or vendored into this repo. This repo contains only my own content, config, and
assets. To update the theme:

```sh
hugo mod get -u        # bump PaperMod (and any other modules) to latest
hugo mod tidy          # prune go.sum
git commit go.mod go.sum
```

## Local development

Requires **Hugo extended ≥ 0.146** and **Go** (for modules).

```sh
brew install hugo go       # macOS
hugo server                # live-reload at http://localhost:1313
hugo server --buildDrafts  # include drafts (e.g. riddler-coins)
```

## Writing posts

Posts live in `content/blog/`. Both Markdown (`.md`) and Org (`.org`) are
rendered natively — no more exporting org → HTML by hand. Front matter for org
files uses org keywords:

```org
#+TITLE: My Post Title
#+DATE: 2025-01-15T00:00:00-08:00
#+SLUG: my-post-title
#+TAGS[]: emacs lisp
#+DRAFT: true
#+OPTIONS: toc:nil num:nil
```

## Customizing the theme (without forking it)

PaperMod is a module, so it isn't in this repo — but you can still override or
extend any part of it. Files in this project win over the module's copies and
are untouched by `hugo mod get -u`:

- `layouts/_partials/header.html` — overrides PaperMod's header to add the
  **Game Reviews** dropdown (parent menu entry + one level of submenu). Only the
  menu loop differs from upstream; a comment at the top points to the theme file
  to diff against when re-syncing.
- `assets/css/extended/dropdown.css` — dropdown styling. PaperMod automatically
  concatenates everything in `assets/css/extended/*.css`, so any CSS tweaks go
  here (use the theme's CSS variables like `--primary`, `--border`, `--theme`).

The nav itself is configured in `hugo.toml` under `[menu]`. "Game Reviews" is a
parent entry (no URL) with Board/Video Games as children. The **Blog** entry is
commented out — the `/blog/` page still builds; uncomment to show it in the nav.

## Deployment

Push to `main`; GitHub Actions (`.github/workflows/hugo.yaml`) builds and
deploys to GitHub Pages. Custom domain `carl.ac` is set via `static/CNAME`.
