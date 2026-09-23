# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static HTML design mockups of the Summer v2 homepage, built from a Figma export. There is no build step, package manager, linter or test suite. Edit the HTML and reload the browser.

This repo is a standalone copy of the `mockups/` folder from the Shopify theme repo. `docs/2026-09-21_summer-v2-mockup-accent-switcher.md` is the detailed record of that work. Its paths say `mockups/…`, but here those files are at the repo root, and theme paths such as `snippets/icon-*.liquid` don't exist in this repo.

GitHub Pages serves the root of `main` at https://wesleyhuang23.github.io/fe-brand-design/. `.nojekyll` makes Pages serve the files as-is. Pushing to `main` deploys the site.

## Preview and verify

```sh
python3 -m http.server 8777   # serve over http, not file://
```

Serve over http because `history.replaceState` throws on `file://`, so the URL-param path of the accent switcher can only be tested over http.

To verify a change, use the `browser-automation` skill (`node ~/.claude/skills/browser-automation/browser.mjs <url> --eval "..."`). It reports console errors and failed requests. Also take a screenshot and look at it. An attribute check once passed while the element was visibly wrong (see "Gotchas" below).

## Architecture

- **Each page is one self-contained file.** CSS and JS are inline, and there are no shared files. This was deliberate, so each mockup can be moved around as a single file. Images load from `summer-v2-assets/` (Figma crops at 2×).
- `summer-v2-homepage.html` is the desktop page, built from the 1440px artboard with breakpoints at 1280, 1080 and 767px. `summer-v2-homepage-mobile.html` is a separate mobile page. On screens wider than 520px it sits in a centered frame up to 430px wide. The mobile header has no wishlist heart.
- Both pages have the same sections in the same order: `.announce`, header, `.hero`, `.discover`, `.feature`, `.arrivals`, `.letter`, `.picks`, footer.
- Design tokens live in the `:root` block of each file. The accent colors are `--mauve`, `--mauve-dark`, `--mauve-soft`, `--teal` and `--teal-script`. Heart SVG `fill` and `stroke` use `var(--mauve…)` / `var(--teal)`, so they follow the accents. `--rust` is declared but unused.
- Fonts: type follows the Figma file "Frank & Eileen 2026" (`LqvLmsxbbYe0EyHVGyjxy3`, desktop node `2:185`, mobile node `2:266`). The fonts are Canela Web Light (`--display`), Geist (`--sans`), Shirley (`--hand`) and Beth Ellen (`--script`).
  - **Shirley** is self-hosted from `fonts/` and declared with an `@font-face` in each page. The files are named `Shirley-Bold`, but the font's family is "Shirley Regular", which is the weight Figma uses, so the `@font-face` maps them to weight 400. Don't substitute a Google font for Shirley.
  - **Canela Web** is self-hosted from `fonts/` too, and always uses **weight 300 (Light)**, the same as the theme (`assets/fonts.scss.liquid` in the Shopify repo). Never Medium or Regular. Light 300 is the **only** face declared, on purpose: asking for a heavier weight makes the browser fake a bold instead of using Canela's own cut. `fonts/Canela-Light-Web.woff` actually holds WOFF2 data despite its name, so its `format()` hint says `woff2` — Safari and Firefox reject it when it says `woff`, and the page then falls back to a heavier serif. `Canela-Regular-Web*` sits in `fonts/` but nothing uses it.
  - Geist and Beth Ellen load from Google Fonts. Nothing loads Cormorant Garamond any more — it stays last in `--display` only as a fallback if a font file fails.
  - `fonts/` must ship with the pages. The published artifacts need these files uploaded as well, or they fall back.
- The logo is `summer-v2-assets/logo.svg`, which combines the Figma wordmark and "EST 1947" vectors into one file.
- `summer-v2-pdp.html` is the desktop product page (Eileen), from Figma node `8:404`. It copies the homepage's header, footer, product-card CSS and accent switcher, and its photos are `summer-v2-assets/pdp-*`. The rating hearts are Figma's vectors, inlined as `<symbol>`s so they follow `--teal`.
- `index.html` is the Pages landing page. It links to all three mockups.

### Accent color switcher (duplicated in every page)

The switcher is a review tool, not part of the design. It runs from the comment `<!-- ========= Accent color switcher (mockup review tool, not part of the design) ========= -->` to `</body>`, and that block is **identical in all three HTML files**. Don't edit only one copy. Edit the block once and re-inject it into every file. The docs file has a Python snippet for this (§3).

- It sets inline styles for the accent variables on `:root`.
- On load it takes state from the URL params first (`?primary=74825e&pop=81a7af`), then localStorage (`fe-mockup-accents`, `fe-mockup-accents-open`), then the file defaults. All pages use the same storage keys.

## Gotchas

- `.fe-accent__custom { display:flex }` beat `[hidden]`, so `#fe-accent-panel [hidden] { display:none !important }` is needed. Assert on the computed `display`, not on `el.hidden`.
- The dropdown menus open **upward** (`bottom: calc(100% + 4px)`) because the panel is pinned to the bottom of the screen. Don't change this to `top`.
- `.btn` has a 0.2s background transition. Wait about 400ms before asserting on button colors.
- If a Figma icon arrives as a base64 raster PNG, don't embed it. Use a vector icon instead.

## Visibility

The docs file notes that public hosting was rejected earlier because these pages contain unreleased collection photography. This repo and its Pages site are public. Check with the user before adding more unreleased assets.
