# Migration guide: Beautiful Jekyll (forked) → Hugo + PaperMod (module)

This is the throwaway build produced on a work machine to de-risk the move.
The canonical copy lives on your personal machine + the GitHub repo. Follow
these steps **on your personal machine** to go live.

---

## What changed and why

| Before | After |
|---|---|
| Forked Beautiful Jekyll — theme + demo content + your content all mixed in one git history | Your content only; **PaperMod pulled in as a Hugo Module** (`go.mod`) |
| Jekyll (Ruby) + GitHub's built-in Node-20 Pages build → hit the deprecation | Hugo (Go binary); custom Actions workflow on current action versions → no Node treadmill |
| org → HTML export by hand, HTML committed to `_posts/` | **Hugo renders `.org` natively**; commit the `.org` file, done |
| Theme updates = merge upstream into fork, resolve conflicts against your content | `hugo mod get -u` — one command, no conflicts |

## Verified on the work machine

A full `hugo --gc --minify` build passed (42 pages, 20 aliases, exit 0). Spot-checked:

- **Every old URL preserved** — `/board-game-reviews/`, `/video-game-reviews/`,
  `/introducing-kdeconnect-el/`, `/emacs-baloo/`, `/five-months-with-rocksmith/`,
  `/4k-monitor/`, `/blogging-with-emacs-org-github-pages/`,
  `/death-bereavement-creativity/` (via `[permalinks]` rule + per-post `#+ALIASES[]`).
- **Org rendering** — headings, `<em>` italics, footnotes (38 refs in the
  Rocksmith post), inline `code`, and syntax-highlighted `#+BEGIN_SRC` blocks
  (emacs-lisp / R) all render correctly, including the ~99 KB video-game-reviews file.
- **Draft handling** — `riddler-coins` (never published in Jekyll) is marked
  `#+DRAFT: true` and excluded from the build. Publish it later by removing that line.
- Homepage profile mode (headshot + bio), CV PDF link, RSS (`/index.xml`), and
  sitemap all OK.
- **Nav** — CV / Research / Teaching / **Game Reviews ▾** (dropdown → Board
  Games, Video Games). Dropdown verified working in light + dark mode. The Blog
  nav link is intentionally commented out (see decision #5); the `/blog/` page
  and all posts still build and are reachable by URL.

---

## Content decisions baked in (change if you disagree)

1. **"Game Reviews" dropdown restored.** Beautiful Jekyll had a nested dropdown
   (Board Games / Video Games). PaperMod's top menu is flat and has no native
   dropdown support, so this is implemented as two small **project-level
   overrides** (not a theme fork — they survive `hugo mod get -u`):
   - `layouts/_partials/header.html` — a copy of PaperMod's header with *only*
     the menu loop changed to render one level of submenu. A comment at the top
     points to the upstream file to diff against if you ever re-sync.
   - `assets/css/extended/dropdown.css` — dropdown styling using the theme's own
     CSS variables (matches light + dark). PaperMod auto-includes anything in
     `assets/css/extended/`, so no template wiring is needed.
   - In `hugo.toml`, "Game Reviews" is a parent `[[menu.main]]` entry (no URL)
     with Board/Video Games as `parent = "gamereviews"` children.
   - **Gotcha fixed:** PaperMod's `.menu` sets `overflow-x: auto`, which clips
     the dropdown panel. `dropdown.css` overrides `#menu { overflow: visible }`
     to release it. (If you ever add many nav items and want horizontal
     scrolling back on mobile, that's the line to reconsider.)
2. **Homepage uses PaperMod "profile mode"** (centered headshot + bio) — closest
   match to your old `index.md`. Config in `hugo.toml` under
   `[params.profileMode]`. (The two default nav buttons were removed as
   redundant with the header.)
3. **Research page PDF links made absolute** (`/research/foo.pdf`) so they
   resolve regardless of trailing slash. PDFs live in `static/research/`.
4. **`defaultTheme = "light"`** to match the current site. Set to `"auto"` in
   `hugo.toml` if you want dark-mode support (PaperMod has it built in).
5. **Blog nav link commented out.** You don't use the blog now but may later.
   The `[[menu.main]]` block for Blog in `hugo.toml` is commented out, so it's
   hidden from the nav — but the `/blog/` listing page and every post still
   build and stay reachable by URL. Uncomment those four lines to show it again.

---

## Steps to go live (on your personal machine)

### 1. Get the toolchain

```sh
brew install hugo go      # Hugo extended + Go (for modules)
hugo version              # confirm "extended", ≥ 0.146
```

### 2. Get these files into your working copy

Copy the contents of this kit over your repo's working tree (or start a fresh
clone and drop them in). The kit is a complete Hugo site:

```
hugo.toml                     # site config (base URL, nav, PaperMod params)
go.mod  go.sum                # PaperMod as a versioned module — COMMIT BOTH
.gitignore
README.md
MIGRATION.md                  # this file
content/
  _index.md                   # homepage (profile mode drives content)
  research.md  teaching.md
  blog/_index.md
  blog/*.org                  # 9 posts (riddler is a draft)
layouts/
  _partials/header.html       # override: adds the Game Reviews dropdown
assets/
  css/extended/dropdown.css   # override: dropdown styling (auto-included)
static/
  CNAME                       # carl.ac  — keeps your custom domain
  headshot.jpg  favicon.ico  carl_lieberman_cv.pdf
  research/*.pdf
.github/workflows/hugo.yaml   # build + deploy
```

> The `layouts/` and `assets/` dirs are how you customize PaperMod *without*
> forking it: files there override or extend the module's copies and are never
> touched by `hugo mod get -u`. That's the whole point of the module approach —
> your changes and the theme stay cleanly separated.

You can delete the old Jekyll files entirely: `_config.yml`, `_layouts/`,
`_includes/`, `_data/`, `_posts/`, `css/`, `js/`, `img/`, `blog/`, `Gemfile*`,
`Dockerfile`, `feed.xml`, `tags.html`, `404.html`, `index.md`, `research.md`
(old), `teaching.md` (old), `CHANGELOG.md`. Keep `LICENSE` if you like. Keep the
`org/` source files if you want, but they're now redundant — the `.org` files in
`content/blog/` are the live source.

> Tip: do this on a branch (`git switch -c hugo-migration`) so you can preview
> before touching `main`.

### 3. Fix the module path if your repo differs

`go.mod` currently declares:

```
module github.com/carldotac/carldotac.github.io
```

If your repo path is different, run `hugo mod init github.com/<you>/<repo>` to
correct it. (It's just an identifier; it doesn't fetch anything.)

### 4. Build and preview locally

```sh
hugo mod get                     # fetch PaperMod into the local module cache
hugo server                      # http://localhost:1313 — click through every page
```

Check the two big review pages and any footnote/code-heavy posts render the way
you want. If a complex org construct looks off (go-org covers most but not
100% of Emacs org), you have two escape hatches per-post:
- wrap the tricky bit in `#+BEGIN_EXPORT html ... #+END_EXPORT` with hand HTML, or
- keep that one post as exported `.html` in `content/blog/` instead of `.org`.

### 5. Point GitHub Pages at Actions

In the repo: **Settings → Pages → Build and deployment → Source = GitHub Actions.**
(Previously it was "Deploy from a branch". This is the switch that stops the old
Node-20 Jekyll build.)

### 6. Commit and push

```sh
git add -A
git commit -m "Migrate to Hugo + PaperMod (theme as module); drop forked Jekyll"
git push origin hugo-migration      # open a PR, or push straight to main when ready
```

On push to `main`, the workflow builds and deploys. Watch the **Actions** tab;
first run takes a couple minutes (it installs Hugo + Go, fetches the module).

### 7. Confirm the domain

`static/CNAME` (contents: `carl.ac`) is emitted to the site root, so the custom
domain carries over. After the first successful deploy, check **Settings → Pages**
still shows `carl.ac` and "Enforce HTTPS" is ticked. DNS at your registrar does
**not** need to change — it still points at GitHub Pages.

---

## Rolling back

If anything's wrong, the old site is still in git history. Revert the migration
commit (or switch Pages Source back to "Deploy from a branch" pointing at a
branch that still has the Jekyll files) and you're back to the old site.

## Cleanup on the work machine (optional)

Nothing personal persists there, but to remove the temporarily installed tools:

```sh
brew uninstall hugo go
rm -rf /tmp/carl-hugo /tmp/carldotac_inspect
```
