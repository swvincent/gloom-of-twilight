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

`README.md` restates several of the facts the checklists below cover — both background dimensions,
the font filenames including Chomsky's `.woff` fallback, and all three icon sizes. Anything that
triggers one of those checklists dates the README too.

## The quotes

Each page's quote appears **twice in its own file**: in the `<blockquote>` and, with the
attribution, in `<meta name="description">`. Changing one and not the other fails silently — a
stale description surfaces only in search results and link previews. The `<title>` does not carry
the quote and does not need touching.

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

Adding or swapping a font is a **four-place change**, and missing any one of them fails silently:

1. the `.woff2` in `fonts/` — Chomsky also ships a `.woff` fallback listed as a second `src`, so
   swap both files or neither
2. an `@font-face` block in `style.css`
3. a `<link rel="preload">` in `index.html`
4. the same `<link rel="preload">` in `404.html`, root-relative

Chomsky is a display face with limited glyph coverage. Any character added to the quote that it
lacks falls back to a plain serif with no error — visually obvious only if you look closely. Check
new text in the browser before considering it done:

```js
// Identical advance width in all three families => Chomsky has no glyph and the
// character is being drawn by a fallback. Paste into the console on the served page.
await document.fonts.load('100px Chomsky');
const c = document.createElement('canvas').getContext('2d');
const w = (family, ch) => { c.font = `100px ${family}`; return c.measureText(ch).width; };
[...'her robes are gloom of twilight...'].filter(ch =>
  w('Chomsky', ch) === w('serif', ch) && w('Chomsky', ch) === w('monospace', ch)
);  // => the characters with no Chomsky glyph
```

`document.fonts.check()` does *not* answer this — with no `unicode-range` descriptor a face claims
to cover everything, so it returns true for characters Chomsky lacks.

## Background image

The background is served in two sizes: `gloom-of-twilight-background.png` (2304×1536) and
`gloom-of-twilight-background-phone.png` (1536×1024) for portrait phones. Both are 16-color
indexed PNGs, which is why 3.5 megapixels fits in 468 KB.

Swapping or resizing either one is a **four-place change**:

1. the `.png` at the repo root
2. the `background-image` in `style.css` — the `body` rule, or the `@media (max-width: 500px)`
   override
3. the matching `<link rel="preload" media="...">` in `index.html`
4. the same `<link rel="preload" media="...">` in `404.html`, root-relative

Each file's two preloads' media conditions must stay exact complements of each other and must match
the `@media` block. Skipping step 3 or 4 fails silently and makes phones download *both* files —
slower than not having the phone variant at all.

Sizing rationale: the image is 3:2 and `background-size: cover` crops the sides on portrait phones,
so the width needed is `1.5 × viewport height`, not viewport width. At the tallest common phone
viewport (~932 CSS px) that is ~1398px, which 1536 clears with headroom.

## Icons

Three files at the repo root, all crops of `gloom-of-twilight-background.png`:

- `favicon.ico` — 16, 32 and 48 in the one file. The browser tab.
- `apple-touch-icon.png` — 180×180. The iOS home screen.
- `icon-512.png` — 512×512. Android add-to-home-screen and bookmark tiles.

Adding or swapping one is a **three-place change**:

1. the file at the repo root
2. the `<link>` in `index.html`
3. the same `<link>` in `404.html`, root-relative

Browsers fetch `/favicon.ico` and `/apple-touch-icon.png` from the root with or without the tags;
the tags only make it deterministic. Dropping a file in without them still works, which is exactly
why steps 2 and 3 get forgotten.

The small and large icons use **different crops on purpose**. `favicon.ico` is cut tight on the
figure, because at 16px anything wider is an indistinct smudge. The two PNGs use a far wider crop —
tree, figure, horizon, shoreline — that only holds together at 180px and up. Re-cutting one to
match the other leaves either an illegible tab icon or an empty-looking home-screen icon.

All three take the same white-point lift (`-level 0%,72%`) before resizing. The painting is
near-black and the figure sits only a few values off its ground; unlifted, the icon has no legible
edge at small sizes. That lift is why the icons read slightly less dark than the page itself.

**The PNGs are quantized with a 256-color ceiling, and the 16-color rule above does not carry over
to them.** The background's dither is invisible at 2304px but becomes coarse speckle once
downscaled to icon size, and 16 colors also drains the blue out of the water. A 256-color ceiling
is visually identical to an unquantized build at a third the weight.

The crops land far under that ceiling on their own — 46 colors in `apple-touch-icon.png`, 64 in
`icon-512.png`. That is the quantizer settling, not a target; don't pad a palette to exactly 256
trying to match.

`icon-512.png` is declared with `sizes="512x512"` so it is not a tab-icon candidate — verified in
Chromium, untested elsewhere.

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

Icons are checked separately, and not by screenshotting the page — browsers cache tab icons hard.
Magnify each size and view it against both a light and a dark tab strip: the icon is close in value
to a dark tab strip and can disappear into it.
