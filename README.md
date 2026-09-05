# Gloom of Twilight

A single static page: an AI-generated twilight landscape as a full-bleed
background, with one line of verse centered over it and its source credited in
the bottom-right corner.

> her robes are gloom of twilight...
>
> — Dead Sea Scrolls (4Q184 1:5)

Live at <https://www.gloomoftwilight.com>.

## Contents

| Path | Purpose |
| --- | --- |
| `index.html` | The entire page. |
| `404.html` | Custom not-found page, same layout with a different quote linking home. |
| `style.css` | All styling. |
| `gloom-of-twilight-background.png` | The background image, AI-generated (2304×1536). |
| `gloom-of-twilight-background-phone.png` | Downscaled background for portrait phones (1536×1024). |
| `fonts/` | Self-hosted webfonts. |
| `CNAME` | Custom domain for GitHub Pages. |
| `.nojekyll` | Tells GitHub Pages to serve the tree as-is instead of running Jekyll. |

No build step, no dependencies, no framework — plain HTML and CSS.

## Fonts

Fonts are served from `fonts/` rather than a CDN, so the page has no
third-party requests and renders identically offline.

Two faces are in use:

- **Chomsky** (`chomsky-webfont.woff2`, with a `.woff` fallback) — blackletter,
  for the quote.
- **Lato** (`lato-400.woff2`) — set uppercase and letterspaced, for the
  attribution. A contemporary sans reads clearly at small sizes where the
  blackletter did not.

## Local development

Serve the folder over HTTP rather than opening `index.html` directly — a
`file://` page loads webfonts under different rules than production:

```bash
python -m http.server 8000
```

Then visit <http://localhost:8000>.

## Deploying to GitHub Pages

1. Push this repository to GitHub.
2. In the repository's **Settings → Pages**, set the source to **Deploy from a
   branch** and pick the default branch with the `/ (root)` folder.
3. Under **Custom domain**, enter `www.gloomoftwilight.com`. This matches the
   committed `CNAME` file; GitHub will re-write that file if you change the
   domain here.
4. At your DNS provider, add a `CNAME` record pointing `www` to
   `<your-github-username>.github.io`. If you also want the apex domain
   `gloomoftwilight.com` to work, add `A` records for it pointing at GitHub's
   Pages IP addresses (listed in GitHub's custom-domain documentation).
5. Once DNS resolves, enable **Enforce HTTPS** in Settings → Pages.
