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
- `summer-v2-pdp-mobile.html` is the mobile product page, from Figma node `8:234`. It copies the mobile homepage's shell (announce, header, `.rail`/`.dots`, footer accordion, switcher) and reuses the desktop PDP's photos, swatches and heart symbols. The button-up strip is its own 4-shirt crop, `pdp-button-up-guide-m.png`.
- `index.html` is the Pages landing page. It has two groups: the original mockups, then a **2.0 Designs** section.

### 2.0 designs

A second round from the Figma file **"Frank & Eileen 3.0 2026"** (same file key `LqvLmsxbbYe0EyHVGyjxy3`). The pages are `home-2-0.html` (node `65:611`) and `home-mobile-2-0.html` (node `65:739`). They are copies of the matching v2 pages with the newer components, and they keep the accent switcher unchanged.

- Desktop 2.0 swaps the nav: 11 links (Shop All … Learn) in Geist SemiBold 12px / `.1em` instead of 5 Shirley links, with Sale in the primary accent.
- Mobile 2.0 puts the announcement bar and footer on a fixed denim `--denim: #496179` instead of the primary accent, and its footer headings are Geist ExtraBold (800).
- `pdp-2-0.html` (node `65:12336`) and `pdp-mobile-2-0.html` (node `65:11236`) are the 2.0 product pages. These are redesigns, not variants: a 6-tile gallery (a 3-slide peek carousel on mobile), the full fabric catalogue of 16 groups and ~76 swatches in 43px cells, pill size chips `XXS–XL`, an editorial image band, and on desktop a 5-card carousel plus a wider 5-column footer. Their photos and swatches live in `summer-v2-assets/pdp20/` and both pages share them.
- Figma's 3.0 nav shows a rust primary (`#9e462d`) and amber pop (`#aa6223`) while its letter and footer still show mauve. The pages keep the mauve/teal defaults; Rust is one click away in the switcher.
- Known quirks in the 3.0 PDP frames, and what the pages do about them: the price reads `$288` in the title but `SELECT SIZE — $268` on the button (the pages use `$288` in both); the Yotpo review block sits 50px right of centre in Figma (the pages centre it); two mobile swatch cells have no image (the pages show the fabric, from the desktop export); the footer mixes Geist and Montserrat (the pages use Geist).

### Accent color switcher (duplicated in every page)

The switcher is a review tool, not part of the design. It runs from the comment `<!-- ========= Accent color switcher (mockup review tool, not part of the design) ========= -->` to `</body>`, and that block is **identical in all four mockup files**. Don't edit only one copy. Edit the block once and re-inject it into every file. The docs file has a Python snippet for this (§3).

- It sets inline styles for the accent variables on `:root`.
- It also drives `--denim`, the third brand tone used by the 2.0 mobile announcement bar and footer and by the PDP's small blue link accents. The rule: while `pop` is at the file default, `--denim` keeps the file's own value (`#496179`, as in Figma); as soon as `pop` changes, `--denim` follows it, so no coloured band is left stranded. Reset restores the denim default.
- On load it takes state from the URL params first (`?primary=74825e&pop=81a7af`), then localStorage (`fe-mockup-accents`, `fe-mockup-accents-open`), then the file defaults. All pages use the same storage keys.

## Gotchas

- `.fe-accent__custom { display:flex }` beat `[hidden]`, so `#fe-accent-panel [hidden] { display:none !important }` is needed. Assert on the computed `display`, not on `el.hidden`.
- The dropdown menus open **upward** (`bottom: calc(100% + 4px)`) because the panel is pinned to the bottom of the screen. Don't change this to `top`.
- `.btn` has a 0.2s background transition. Wait about 400ms before asserting on button colors.
- If a Figma icon arrives as a base64 raster PNG, don't embed it. Use a vector icon instead.

## Visibility

The docs file notes that public hosting was rejected earlier because these pages contain unreleased collection photography. This repo and its Pages site are public. Check with the user before adding more unreleased assets.
