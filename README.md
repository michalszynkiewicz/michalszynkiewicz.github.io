# michalszynkiewicz.dev

Personal site + blog. Static, built with **[Hugo](https://gohugo.io)** (a single Go binary —
no `node_modules`, no gem tree, no plugins). Hosted on GitHub Pages at
**https://michalszynkiewicz.dev**.

Design is hand-written (`layouts/` + `assets/css/main.css`), no theme. Light + dark,
mobile-first, no trackers.

---

## Run it locally

Hugo (extended) is already installed at `~/.local/bin/hugo`. Either call it by full path,
or add it to your `PATH` once:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

Then, from the repo root:

```bash
hugo server -D        # -D = include drafts (e.g. the benchmark post)
```

Open **http://localhost:1313/**. The server live-reloads on every save.

If you ever need to (re)install Hugo, grab the **extended** build, version `0.163.3`
(matches CI), from <https://github.com/gohugoio/hugo/releases>.

### Build the production site (what CI does)

```bash
hugo --gc --minify --cleanDestinationDir   # outputs to ./public (git-ignored)
```

(`--cleanDestinationDir` removes stale files from previous builds. CI always builds
from a clean checkout, so this only matters for repeated local builds. The `hugo server`
preview renders in memory and is always fresh.)

---

## Add a blog post

Drop a Markdown file in `content/blog/`. That's the whole workflow.

```bash
hugo new content blog/my-post-title.md   # or just create the file by hand
```

Front matter:

```yaml
---
title: "My post title"
description: "One sentence — used for the listing, SEO meta, and link previews."
date: 2026-07-01
draft: false        # true = won't appear in production (visible with `hugo server -D`)
---
```

Then write Markdown. You get for free:

- **Code blocks** with syntax highlighting (light + dark) — ```` ```python ```` etc.
- **Tables** (GitHub-flavored Markdown).
- **Charts/images** — drop an SVG/PNG in `static/img/` and reference it:
  `![alt text](/img/chart.svg "Optional caption")`. A standalone image with a title
  becomes a `<figure>` with a caption automatically.
- **Footnotes**, blockquotes, `<kbd>` keys, callouts (`<div class="callout">…</div>`).

See `content/blog/benchmarking-coding-agents.md` for a worked example of all of the above
(it's a `draft` scaffold — replace the placeholder numbers and flip `draft: false`).

---

## Publish

Deployment is automatic via GitHub Actions (`.github/workflows/deploy.yml`):

```bash
git add -A && git commit -m "…" && git push     # to master → builds & deploys
```

**One-time GitHub setup:** in the repo, go to **Settings → Pages → Build and deployment**
and set **Source = GitHub Actions**. (Not "Deploy from a branch".)

The workflow installs the pinned Hugo binary, builds, and publishes. Watch runs under the
repo's **Actions** tab.

---

## Custom domain: `michalszynkiewicz.dev`

The repo already contains `static/CNAME` (→ `michalszynkiewicz.dev`), which Hugo copies into
the build. To make the domain live:

### 1. DNS — at your registrar, add these records

**Apex (`michalszynkiewicz.dev`) — A + AAAA records pointing at GitHub Pages:**

| Type | Name | Value |
|------|------|-------------------|
| A    | `@`  | `185.199.108.153` |
| A    | `@`  | `185.199.109.153` |
| A    | `@`  | `185.199.110.153` |
| A    | `@`  | `185.199.111.153` |
| AAAA | `@`  | `2606:50c0:8000::153` |
| AAAA | `@`  | `2606:50c0:8001::153` |
| AAAA | `@`  | `2606:50c0:8002::153` |
| AAAA | `@`  | `2606:50c0:8003::153` |

**Optional `www` → redirects to the apex:**

| Type  | Name  | Value |
|-------|-------|-------|
| CNAME | `www` | `michalszynkiewicz.github.io.` |

### 2. GitHub — Settings → Pages

- Set **Custom domain** to `michalszynkiewicz.dev` and save (this triggers DNS validation).
- Once it validates, tick **Enforce HTTPS**.

> `.dev` is on the HSTS preload list, so HTTPS is **mandatory** — `http://` won't load at all.
> GitHub auto-provisions a Let's Encrypt certificate after DNS validates (can take up to an
> hour). Until then the domain may show a cert warning; that's expected.

---

## Things to fill in

These are centralized in `hugo.toml` under `[params]`:

- `email` — currently `michal.l.szynkiewicz@gmail.com`; change if you want a different contact address.
- `goatcounter` — empty (analytics off) until you set it; see **Analytics** below.

Other content lives in:

- `content/_index.md` — landing hero copy + bio.
- `data/proof.yaml` — the three proof points on the landing page.

---

## Analytics

Visitor counting uses **[GoatCounter](https://www.goatcounter.com)** — privacy-friendly,
no cookies, no IP stored, so no cookie-consent banner is required.

1. Register a site code at <https://www.goatcounter.com> (e.g. `michalsz`).
2. In `hugo.toml`, set `goatcounter = "https://YOURCODE.goatcounter.com/count"`.
3. Push. The snippet loads **only in production builds** — your local `hugo server`
   previews are never counted.

**Your dashboard** (visits, countries, referrers, top pages) lives at
**`https://YOURCODE.goatcounter.com`** — log in with the account you registered. It's
private by default; you can optionally make it public in GoatCounter's settings.

Set `goatcounter = ""` to turn it off.

## Customizing the look

- **Colors / type / spacing:** the CSS custom properties at the top of
  `assets/css/main.css` (`:root { … }` and the dark-mode `@media` block). Change
  `--accent` to re-tint the whole site.
- **Code highlighting theme:** regenerate the Chroma block at the bottom of `main.css`:
  ```bash
  hugo gen chromastyles --style=github       # light
  hugo gen chromastyles --style=github-dark  # dark (wrap in @media (prefers-color-scheme: dark))
  ```
- **Share image:** `static/img/og-default.png` (1200×630). Source HTML used to render it is
  not committed; re-render any 1200×630 image to replace it.
