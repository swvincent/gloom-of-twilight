# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Committing

**Never commit unless explicitly asked.** Leave finished work in the working tree so it can be
previewed in a browser first. Show what changed and wait — "looks good" is not a request to commit.

## What this is

Two static pages (`index.html` and `404.html`, sharing `style.css`) published to GitHub Pages at
`www.gloomoftwilight.com`: a full-bleed AI-generated image with a blackletter quote centered over
it and its attribution in the bottom-right corner.

`404.html` is the same layout with a different quote, wrapped in a link back to `/`. Pages serves
it for a missing path at *any* depth, so every asset reference in it is root-relative (`/style.css`,
`/fonts/...`, `/gloom-of-twilight-background.png`) — relative paths would 404 alongside it. Changes
to the head of `index.html` usually need mirroring there, in root-relative form.

There is no build step, no dependencies, no framework, and no test/lint/build commands. Do not add
a toolchain unless asked.

## Running it

```bash
python -m http.server 8000
```

Serve over HTTP rather than opening `index.html` directly — `file://` loads webfonts under
different rules than production, so font problems will not reproduce.

## Deployment coupling

GitHub Pages serves the repo root verbatim. `CNAME` and `.nojekyll` are load-bearing, not
boilerplate: `CNAME` sets the live domain, and `.nojekyll` stops Pages from running Jekyll over the
tree. Anything committed at the root is published.

## Fonts

Self-hosted in `fonts/` on purpose — the page makes zero third-party requests. Chomsky (blackletter)
sets the quote; Lato sets the attribution, uppercase and letterspaced. Unused font files have been
deleted rather than kept around.

Adding or swapping a font is a **three-place change**, and doing two of the three fails silently:

1. the `.woff2` in `fonts/`
2. an `@font-face` block in `style.css`
3. a `<link rel="preload">` in `index.html`

Chomsky is a display face with limited glyph coverage. Any character added to the quote that it
lacks falls back to a plain serif with no error — visually obvious only if you look closely. Check
new text in the browser before considering it done:

```js
// per character: identical advance width in all three families => no Chomsky glyph
const c = document.createElement('canvas').getContext('2d');
c.font = '100px Chomsky';   const a = c.measureText(ch).width;
c.font = '100px serif';     const b = c.measureText(ch).width;
c.font = '100px monospace'; const d = c.measureText(ch).width;
```

## Background image

The background is served in two sizes: `gloom-of-twilight-background.png` (2304×1536) and
`gloom-of-twilight-background-phone.png` (1536×1024) for portrait phones. Both are 16-color
indexed PNGs, which is why 3.5 megapixels fits in 468 KB.

Swapping or resizing either one is a **three-place change**:

1. the `.png` at the repo root
2. the `background-image` in `style.css` — the `body` rule, or the `@media (max-width: 500px)`
   override
3. the matching `<link rel="preload" media="...">` in `index.html`

The two preloads' media conditions must stay exact complements of each other and must match the
`@media` block. Skipping step 3 fails silently and makes phones download *both* files — slower
than not having the phone variant at all.

Sizing rationale: the image is 3:2 and `background-size: cover` crops the sides on portrait phones,
so the width needed is `1.5 × viewport height`, not viewport width. At the tallest common phone
viewport (~932 CSS px) that is ~1398px, which 1536 clears with headroom.

## Layout constraints

`style.css` has **no layout breakpoints**. All sizing is `clamp()` against viewport
width, and the layout is deliberately identical at every screen size — the attribution stays
bottom-right on phones too. A previous narrow-screen media query that relocated it was removed
because the inconsistent placement made it look missing. Reach for a `clamp()` adjustment before a
breakpoint.

There is exactly one media query in the file, at 500px, and it swaps only the background image
file — no sizing, spacing, or positioning changes inside it. See "Background image" above.

Two things are easy to break:

- The attribution must stay on **one line down to a 320px viewport**. Any change to its text, size,
  or tracking needs rechecking there.
- `figcaption`'s negative `margin-right` exactly cancels its `letter-spacing` so the line sits flush
  with the right gutter. If the tracking changes, that value must change with it.

## Verifying changes

Screenshot at desktop width, then check 320, 375, 412, 768, and a short landscape viewport (e.g.
667x375). Confirm at each: the attribution is on one line and fully on-screen, it does not overlap
the wrapped quote, and neither axis scrolls.
