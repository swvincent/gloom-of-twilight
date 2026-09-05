# Gloom of Twilight

Two static pages sharing one layout: an AI-generated twilight landscape as a
full-bleed background, with one line of verse centered over it and its source
credited in the bottom-right corner.

> her robes are gloom of twilight...
>
> — Dead Sea Scrolls (4Q184 1:5)

`404.html` is that same layout with a different line, wrapped in a link home.

Live at <https://www.gloomoftwilight.com>.

## Contents

| Path | Purpose |
| --- | --- |
| `index.html` | The entire page. |
| `404.html` | Custom not-found page, same layout with a different quote linking home. |
| `style.css` | All styling. |
| `gloom-of-twilight-background.png` | The background image, AI-generated (2304×1536). |
| `gloom-of-twilight-background-phone.png` | Downscaled background for portrait phones (1536×1024). |
| `favicon.ico` | Browser tab icon, holding 16, 32 and 48 px in one file. |
| `apple-touch-icon.png` | 180×180 icon for the iOS home screen. |
| `icon-512.png` | 512×512 icon for Android home screens and bookmark tiles. |
| `fonts/` | Self-hosted webfonts (see below). |
| `CNAME` | Custom domain for GitHub Pages. |
| `.nojekyll` | Tells GitHub Pages to serve the tree as-is instead of running Jekyll. |
| `CLAUDE.md` | Notes for Claude Code on the constraints this repo is easy to break. |

No build step, no dependencies, no framework — plain HTML and CSS.

## Text

Both quotes come from *The Dead Sea Scrolls: A New Translation* by Michael
Wise, Martin Abegg Jr. and Edward Cook (HarperSanFrancisco, 1996). The source
is 4Q184, a wisdom poem from Qumran Cave 4 commonly titled "The Wiles of the
Wicked Woman" — line 1:5 on `index.html`, line 1:1 on `404.html`.

## Fonts

Fonts are served from `fonts/` rather than a CDN, so the page has no
third-party requests and renders identically offline.

Two faces are in use:

- **Chomsky** (`chomsky-webfont.woff2`, with a `.woff` fallback) — blackletter,
  for the quote.
- **Lato** (`lato-400.woff2`) — set uppercase and letterspaced, for the
  attribution. A contemporary sans reads clearly at small sizes where the
  blackletter did not.

## Icons

The three icons at the repository root are all crops of the background image,
each given the same white-point lift before resizing — the painting is close to
black, and without the lift an icon has no legible edge at small sizes.

The crops deliberately differ. `favicon.ico` is cut tight on the figure, since
at 16 px anything wider is an indistinct smudge; `apple-touch-icon.png` and
`icon-512.png` use a much wider crop — tree, figure, horizon, shoreline — that
only holds together at 180 px and up.

Browsers request `/favicon.ico` and `/apple-touch-icon.png` from the root
whether or not the `<link>` tags are present. The tags are there to make it
deterministic, which is why replacing an icon file means editing both
`index.html` and `404.html` as well.

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
