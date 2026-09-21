# fe-brand-design

Static HTML mockups of the Summer v2 homepage, built from the Figma export.

**Live site:** https://wesleyhuang23.github.io/fe-brand-design/

## Pages

- [Desktop](https://wesleyhuang23.github.io/fe-brand-design/summer-v2-homepage.html): `summer-v2-homepage.html`
- [Mobile](https://wesleyhuang23.github.io/fe-brand-design/summer-v2-homepage-mobile.html): `summer-v2-homepage-mobile.html`

## Structure

```
index.html                       Landing page linking to both mockups
summer-v2-homepage.html          Desktop mockup (1440px artboard)
summer-v2-homepage-mobile.html   Mobile mockup
summer-v2-assets/                Photos and badges, cropped at 2×
docs/                            Notes on the mockup work
```

## Local preview

No build step. Open any HTML file in a browser, or serve the folder:

```sh
python3 -m http.server 8000
```

Then visit http://localhost:8000.

## Fonts

The pages use the brand fonts (Canela, Harmonia Sans) when they're installed locally. Otherwise, Google Fonts stand-ins load instead.

## Deploying

GitHub Pages serves the root of `main`. Push to `main` and the site updates within a minute or two.
