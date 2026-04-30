# KCLEA Logo Files

Drop your logo files into this folder using the exact filenames below. The site is pre-wired to pick them up.

## Required (at minimum)

| Filename | What it is | Used where |
|---|---|---|
| `kclea-logo.svg` | **Primary logo** — full-colour, transparent background. Vector preferred. | Header lockup, white-on-light surfaces |
| `kclea-logo-white.svg` | **White / reverse** version of the same logo for dark surfaces. Vector preferred. | Footer (dark band) |

## Nice to have (better quality on mobile + sharing)

| Filename | What it is | Notes |
|---|---|---|
| `kclea-logo.png` | PNG fallback for the primary logo | High-res — at least 1000px on the long edge |
| `kclea-logo-white.png` | PNG fallback for the white version | Same as above |
| `kclea-mark.svg` | Just the symbol / icon (no wordmark) | For favicons, social cards, small spaces |
| `kclea-mark.png` | PNG fallback for the mark | At least 1000×1000px |

## Favicon (separate folder: `../favicon/`)

The favicon set lives in `assets/favicon/`:

| Filename | Size | Notes |
|---|---|---|
| `favicon.ico` | 32×32 (multi-res) | The classic — covers older browsers |
| `favicon-16.png` | 16×16 | Standard browser tab |
| `favicon-32.png` | 32×32 | Retina browser tab |
| `apple-touch-icon.png` | 180×180 | iOS home-screen icon |
| `android-chrome-192.png` | 192×192 | Android home-screen icon |
| `android-chrome-512.png` | 512×512 | Larger Android |

You can generate this whole set from one square PNG (≥ 512×512) using https://realfavicongenerator.net/.

## What happens after you drop files

1. **Drop the file(s) above in `assets/logos/`** (and favicons in `assets/favicon/`).
2. Let me know which files you saved.
3. I'll swap the current text-based "King's / Engineers" red square placeholder for an `<img>` referencing your real logo — header, footer, and favicon `<link>` tags.
4. The fallback chain (SVG → PNG) is built into the `<picture>` element, so older browsers still render PNG cleanly.

## Where the logo currently appears

- **Header lockup** (every page) — the small red square at the top-left
- **Footer brand block** (every page) — the same lockup, against the dark footer
- **Browser tab** (favicon) — currently no favicon
