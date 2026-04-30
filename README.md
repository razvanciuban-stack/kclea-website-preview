# KCLEA — Website Redesign Preview

Visual preview build of the King's College London Engineers' Association website redesign.

> ⚠️ **This is a preview build, not the live site.**
> The original site lives at <https://www.kclea.org.uk/>.
> Search engines are blocked here via `robots.txt` and `<meta name="robots" content="noindex">`.

## What's in here

```
index.html              — single-file SPA preview with hash routing (#home, #events, …)
robots.txt              — disallow all crawlers
assets/
├── images/             — photography (Strand campus, etc.)
├── logos/              — KCLEA primary + white logos
└── favicon/            — favicon set (TBD)
docs/
└── superpowers/specs/  — design specifications per page
```

## Pages

All pages share one HTML file in this preview. Each section is shown/hidden via a small
JS router based on the URL hash:

| Hash | Page |
|---|---|
| `#home` | Homepage |
| `#events` | Events index |
| `#event-hardhacks26` | Hardware Hacks 26 detail |
| `#bulletin` | The Bulletin |
| `#annual-lunch` | Annual Lunch 2026 |
| `#annual-lecture` | Annual Lecture 2026 |
| `#annual-lecture-past` | Past Annual Lectures (archive) |
| `#committee` | Committee |
| `#about` | About KCLEA |
| `#about-membership` / `#about-bursaries` / `#about-constitution` / `#about-13club` / `#about-directory` / `#about-links` | About sub-pages |
| `#in-memoriam` | In Memoriam |
| `#contact` | Contact form |

In production, each of these will become a separate `.html` file with normal `<a href="…">`
navigation — no JavaScript routing required.

## Running locally

```bash
# From the project root
python3 -m http.server 8080
# then open http://localhost:8080
```

## Brand

- **Display type:** Cormorant Garamond (closest free web equivalent to King's Caslon)
- **UI / body type:** Inter
- **Primary colour:** King's red `#D3231A`
- **Reference:** [Rose Design KCL Brand Identity](https://www.rosedesign.co.uk/work/kcl-brand-identity/)

## Going public

Before launch:

1. Remove the `<meta name="robots" content="noindex…">` lines in `index.html`
2. Replace `robots.txt` with the public-friendly version
3. Generate the favicon set in `assets/favicon/` (e.g. via [realfavicongenerator.net](https://realfavicongenerator.net/))
4. Split `index.html` into per-page files (`events.html`, `bulletin.html`, etc.) — each linking
   normally with `<a href="…">` rather than the current hash router
