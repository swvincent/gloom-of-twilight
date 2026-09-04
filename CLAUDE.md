# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Committing

**Never commit unless explicitly asked.** Leave finished work in the working tree so it can be
previewed in a browser first. Show what changed and wait — "looks good" is not a request to commit.

## What this is

A single static page (`index.html` + `style.css`) published to GitHub Pages at
`www.gloomoftwilight.com`: a full-bleed painting with a blackletter quote centered over it and its
attribution in the bottom-right corner. `mockup.png` is the gitignored design reference.

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

## Layout constraints

`style.css` has **no media queries and no breakpoints**. All sizing is `clamp()` against viewport
width, and the layout is deliberately identical at every screen size — the attribution stays
bottom-right on phones too. A previous narrow-screen media query that relocated it was removed
because the inconsistent placement made it look missing. Reach for a `clamp()` adjustment before a
breakpoint.

Two things are easy to break:

- The attribution must stay on **one line down to a 320px viewport**. Any change to its text, size,
  or tracking needs rechecking there.
- `figcaption`'s negative `margin-right` exactly cancels its `letter-spacing` so the line sits flush
  with the right gutter. If the tracking changes, that value must change with it.

## Verifying changes

Screenshot against `mockup.png` at desktop width, then check 320, 375, 412, 768, and a short
landscape viewport (e.g. 667x375). Confirm at each: the attribution is on one line and fully
on-screen, it does not overlap the wrapped quote, and neither axis scrolls.
