# Summer v2 Mockups — Accent Switcher & Header Icons

**Date:** 2026-09-21
**Scope:** `mockups/summer-v2-homepage.html`, `mockups/summer-v2-homepage-mobile.html`
**Branch worked on:** `feid/dev/1417`
**Status:** Done and verified. Left uncommitted in the working tree.

> **`mockups/` is gitignored.** The `.gitignore` entry `mockups` means none of the work
> described here is tracked by git, on any branch. This document is the only tracked record
> of it. A fresh clone will not have those files — see §7 before assuming they are missing
> because someone deleted them.

---

## 1. What this is

`mockups/` holds two standalone HTML design mocks built from a Figma export of the Summer v2
homepage. They are **not part of the theme or the webpack build** — no `yarn dev`, no
`yarn watch`, no Shopify push. Edit the HTML and reload the browser.

Photos live in `mockups/summer-v2-assets/` (~3 MB), extracted from the Figma SVG at 2×.

Three things were added on 2026-09-21:

1. A floating **accent color switcher** for previewing seasonal palettes.
2. The hand-drawn **heart** and **bag** icons in the header.
3. Heart SVGs rewired to follow the accent CSS variables.

---

## 2. The accent switcher

A fixed button, bottom-left, that opens a panel with one dropdown per accent. Each dropdown
lists the seasonal palette from Figma plus a `Custom…` option that reveals a color/hex input.

### Which variables it drives

Set as inline styles on `:root`, so they override the `:root` block in the file:

| Variable | Source | Used by |
|---|---|---|
| `--mauve` | primary | announce bar, `.btn`, footer, `.nav a.is-sale`, letter greeting |
| `--mauve-dark` | primary darkened 12% | `.btn:hover` |
| `--mauve-soft` | primary lightened 18% | signoff hearts (desktop) |
| `--teal` | pop | `.btn--teal`, search underline, mobile signoff hearts |
| `--teal-script` | pop | hero kicker |

`--mauve-soft` was **added** to the `:root` block in both files on this date; it did not
exist before. `--rust` is declared but unused — leave it or delete it, nothing reads it.

### The palette

Taken from the Figma "Seasonal Accents" board. The first two are the mock's existing colors.

| Name | Hex | Tag |
|---|---|---|
| Mauve | `#98685F` | current |
| Teal | `#537A82` | current |
| Green | `#74825E` | Aug |
| Rose | `#9D6B62` | Aug |
| Rust | `#9E462D` | Sept |
| Tan | `#E2A671` | Sept |
| Blue | `#81A7AF` | Jan |

Both dropdowns offer all seven, so any color can go in either slot. Oct was empty in Figma.

> The Figma board labels **both** Aug swatches `accent/primary`, while Sept and Jan are
> primary + pop. That looks like a slip in the source file, not an intentional rule. It was
> not resolved with the designer — if a month preset feature gets built later, confirm it first.

### State and links

Priority on load: **URL params → localStorage → file defaults.**

- URL: `?primary=74825e&pop=81a7af` (hex without `#`)
- localStorage key `fe-mockup-accents`, value `"#74825e,#81a7af"`
- Key `fe-mockup-accents-open` remembers whether the panel was open
- **Reset** restores the file defaults, clears storage, and strips the query params

Desktop and mobile share the storage keys, so they stay in sync in one browser.

---

## 3. Editing the switcher — read this first

**The entire block is duplicated verbatim in both HTML files.** There is no shared JS file;
that was a deliberate call so each mock stays a single portable file you can drag anywhere.

Do **not** hand-edit the block in one file and hope. Extract it, edit once, re-inject into both:

```python
# from mockups/
import io
MARK = '  <!-- ========= Accent color switcher (mockup review tool, not part of the design) ========= -->'

# 1. extract to a scratch file, edit that
s = io.open('summer-v2-homepage.html', encoding='utf-8').read()
io.open('/tmp/block.html', 'w', encoding='utf-8').write(s[s.index(MARK):s.index('</body>')])

# 2. re-inject into both
BLOCK = io.open('/tmp/block.html', encoding='utf-8').read()
for name in ('summer-v2-homepage.html', 'summer-v2-homepage-mobile.html'):
    s = io.open(name, encoding='utf-8').read()
    assert s.count(MARK) == 1 and s.count('</body>') == 1, name
    io.open(name, 'w', encoding='utf-8').write(s[:s.index(MARK)] + BLOCK + s[s.index('</body>'):])
```

The block runs from that comment marker to `</body>` and is self-contained: `<style>`,
markup, `<script>`, all inline. Nothing outside it references it except the `--mauve-soft`
declaration in `:root`.

---

## 4. Gotchas that cost time — do not re-discover these

**`[hidden]` loses to an author `display`.** `.fe-accent__custom { display: flex }` overrode
the UA stylesheet's `[hidden] { display: none }`, so both custom hex rows rendered on load
despite having the attribute set. Guarded now by `#fe-accent-panel [hidden] { display: none !important; }`.

> This one passed a green test. The assertion read `el.hidden`, which was correctly `false`,
> while the element sat visible on screen. **Assert on computed `display`, not the attribute.**
> It was only caught by taking a screenshot and looking at it.

**Menus open upward.** The panel is pinned to `bottom: 22px`, so a dropdown opening downward
renders below the viewport — the pop dropdown was literally unclickable. `.fe-accent__menu`
uses `bottom: calc(100% + 4px)`. Don't "fix" it back to `top`.

**`replaceState` throws on `file://`.** Opaque origin, so the URL rewrite fails when the file
is opened by double-clicking. It's wrapped in try/catch, and the **Copy link** button is the
path that always works there. Over http (`python3 -m http.server`) the address bar updates live.

**Transitions fool a fast probe.** `.btn` has `transition: background .2s`. Reading
`backgroundColor` 150 ms after a change returns an in-between value and looks like a bug.
Allow ~400 ms before asserting on any `.btn` color.

---

## 5. Header icons

| Icon | Source | Class |
|---|---|---|
| Heart (wishlist) | hand-drawn vector supplied by Wesley from Figma | `.icon--heart` — 21×22, `stroke-width: .9` |
| Bag (cart) | **`snippets/icon-cart.liquid`** — the theme's own icon | `.icon--bag` — 19×22, `fill: currentColor; stroke: none` |

Both override parts of `.icon`, which sets `fill: none; stroke: currentColor; stroke-width: 1.2`.
The bag is filled rather than stroked, hence the explicit opt-out. 19×22 preserves the
104.57 × 120.61 artboard ratio next to the 21×22 heart.

**Desktop only for the heart** — the mobile header has no wishlist icon, just the bag.

> A Figma export of the bag was tried first and abandoned. It was a 547×551 **raster PNG**
> embedded as base64 inside `<pattern>`/`<image>`, tens of thousands of characters long.
> Beyond being impractical to transcribe, a bitmap can't inherit `currentColor` and would
> have added ~26 KB to each file. The theme's vector is strictly better. If another Figma
> icon arrives as a raster export, **check `snippets/icon-*.liquid` first** — the theme has
> ~90 of them.

### Heart fills now follow the accents

| File | Element | Was | Now |
|---|---|---|---|
| desktop | `.letter__hearts` | `#a0452f` | `var(--mauve)` |
| desktop | signoff motto | `#b07a70` | `var(--mauve-soft)` |
| mobile | first heart pair | `#98685f` | `var(--mauve)` |
| mobile | signoff motto | `#537a82` | `var(--teal)` |

Set via the `fill`/`stroke` presentation attributes, which accept `var()` in all current browsers.

---

## 6. Verifying a change

There is no test suite. Use the `browser-automation` skill (headless, reports console errors
and failed requests):

```bash
cd mockups && python3 -m http.server 8777 &
node ~/.claude/skills/browser-automation/browser.mjs \
  "http://localhost:8777/summer-v2-homepage.html?primary=74825e&pop=81a7af" \
  --eval "new Promise(r=>setTimeout(()=>r({
     mauve: getComputedStyle(document.documentElement).getPropertyValue('--mauve').trim(),
     announce: getComputedStyle(document.querySelector('.announce')).backgroundColor
   }),1200))"
```

Serve over http rather than `file://` — it's the only way to exercise the URL-param path.

What was covered on 2026-09-21, both files, 0 console errors and 0 failed requests: dropdown
selection, the `Custom…` flow, outside-click and Escape to close, reload persistence, Reset,
and URL-param entry. Plus screenshots of the panel and the icon row, because of §4.

---

## 7. Open items

**`mockups/` is gitignored and `mockups.zip` is stale.** The zip at the repo root predates all
of this. Re-zip before handing the mocks to anyone.

**The published artifact is at version 4:**
`https://claude.ai/artifact/NNd7zM6e5V5hZxNkT9UWHD` — desktop only; the mobile mock is not
published anywhere.

To update it: **read it first**, then publish with `url` set to that link. Publishing without
`url` creates a *separate* artifact instead of updating this one.

> The published copy is **not** byte-identical to the local file. It carries a `.mock-label`
> badge ("Design mock · not the live site") — CSS plus one `<div>` before the announcement bar —
> that the local file does not have. Re-add it when rebuilding the publish copy, or the badge
> silently disappears from the artifact.

**GitHub Pages was investigated and is not viable.** `gooddog/frank-eileen-shopify` is private
and the `gooddog` org is on the **free** plan — Pages requires Pro/Team/Enterprise for private
repos. More decisively, a Pages site is *public* at any plan below Enterprise Cloud, which
would expose unreleased collection photography. A separate public repo would have the same
problem. Cloudflare Pages or Netlify with access control were suggested as alternatives if a
real URL is needed; **no decision was made and nothing was set up.** The private artifact link
is the current sharing route.
