# KCLEA Homepage Redesign — Design Spec

**Date:** 2026-04-29
**Author:** brainstormed with Claude
**Source:** existing site at https://www.kclea.org.uk/index.html
**Brand reference:** King's College London identity by Rose Design (https://www.rosedesign.co.uk/work/kcl-brand-identity/)
**Mockup:** `.superpowers/brainstorm/4296-1777452322/homepage-v1.html`

---

## 1. Context

KCLEA is the alumni association for King's College London engineers. The current site (built on Squarezone Club Sites) has dated visual styling: solid red navigation bar, plain Helvetica-style sans-serif, six identical red header bars over text blocks, no typographic hierarchy, no visual identity beyond the red color. The committee wants a visual refresh that reads as a contemporary alumni publication while keeping all existing content.

## 2. Goals

- **Pure visual modernization** — same content, same site structure, new look. No new functionality.
- **Honour the King's brand** — apply King's Red, a Caslon-style display serif, and a square red lockup that echo KCL's Rose-Design identity.
- **Improve readability** — replace the flat 6-block grid with a magazine-style hierarchy where the hero, current call-to-action, and primary content each have distinct visual weight.
- **Preserve all original copy** — no content edits beyond very light punctuation and link rewording for clarity.

## 3. Scope

**In scope (this spec):** The homepage at `/` (or `/index.html`).

**Out of scope (this spec):** Inner pages (Events, Bulletin, Annual Lunch, Annual Lecture, Committee, About Us, In Memoriam, Contact, Webmaster). The user has stated those will be addressed in subsequent specs once the homepage direction is approved.

## 4. Visual Language

### 4.1 Typography

- **Display / headlines:** *Cormorant Garamond* — variable Caslon-style serif, free via Google Fonts. Closest free web equivalent to King's Caslon (custom Dalton Maag typeface used by KCL). Weights 400–700, italic styles 400–600.
- **UI / body:** *Inter* — modern grotesque, free via Google Fonts. Weights 400–700.
- **Pairing rule:** All section titles, card titles, and the primary headline in Cormorant Garamond. Italic accents in red within headlines (e.g. *"For King's engineers, past and present"* renders the italic word in #D3231A). Body copy, navigation, captions, and small UI labels in Inter.

### 4.2 Colour

| Token | Hex | Use |
|---|---|---|
| `--kings-red` | `#D3231A` | Primary brand colour. Logo block, italic accents in headlines, kickers, link underlines, primary buttons. |
| `--ink` | `#0d0d0d` | Default text colour, dark band backgrounds, secondary CTA. |
| `--ink-muted` | `#3a3a3a` | Body copy. |
| `--ink-soft` | `#5a5a5a` | Secondary copy, captions. |
| `--paper` | `#ffffff` | Page background. |
| `--paper-warm` | `#fafaf7` | Alt section background (Stay-connected band). |
| `--rule` | `#ececec` | Borders, grid gaps. |
| `--footer-ink` | `#0d0d0d` | Footer background. |
| `--footer-text` | `#bcbcbc` | Footer body text. |

The KCL guidelines list a wide accent palette (Pearl Grey, Slate Grey, Teal Blue, Pea Green, Hot Pink, Powder Blue, Midnight Blue, Pantone Reflex Blue, Sky Blue, Soft Pink, Sea Blue, Lime Green, Orange, Lilac, Purple, Lime Green 2, Yellow). The homepage does **not** use those — it stays disciplined on red + ink + paper. Accent colours are reserved for inner pages or future campaign moments.

### 4.3 Lockup

The committee has a new KCLEA logo in production but it is not yet ready. **Until the new logo is delivered, the homepage uses a text-based placeholder lockup**:

A square red block containing two lines of Cormorant Garamond white type:

- Line 1: *"King's"* (italic, 14px, 600)
- Line 2: "Engineers" (uppercase, 9px Inter, .18em letter-spacing)

This echoes the KCL square crest pattern Rose Design established. Used in the header (paired with the wordmark "KCLEA / King's College London Engineers' Association") and again in the footer.

**Swap path when the new logo arrives:**
- Replace the inner two-line text with an `<img>` (SVG preferred) of the new mark, sized to fit the same red square box (≈ 40×40px in the header).
- The wordmark beside it ("KCLEA / King's College London Engineers' Association") and all other components remain unchanged.
- The implementation should isolate the lockup as a single small partial / template fragment so the swap is one file edit.

### 4.4 Layout & Spacing

- 32px horizontal page padding (28px on the brainstorm preview shell, will read as 64px on a real ~1200px viewport with content max-width of ~1100px).
- Sections separated by 1px `--rule` borders, no shadows.
- Grid gaps use 1px `--rule` between cells (events grid, six-card grid) for a "newspaper rule" look rather than card shadows.
- Generous vertical rhythm: 64px top/bottom padding on each major section.

### 4.5 Interaction (hover & focus)

Every interactive element has a visible state change. Transitions are short (150–250ms ease) and never affect layout box size.

| Element | Default | Hover / focus |
|---|---|---|
| Header nav links | Ink colour, no underline | King's red colour + 1px red underline animating from left (`scaleX 0 → 1`, 250ms) |
| Active nav item | King's red, 600 weight, full underline | (no change — already at hover state) |
| Header CTA "Join KCLEA" | Black pill, white text | King's red pill, white text, lifted 1px (`translateY(-1px)`) |
| Hero meta-line links | Red, faint underline | Solid red underline + faint red wash background (`#D3231A14`) |
| Hardware Hack white CTA | White pill, ink text | King's red pill, white text, lifted 1px |
| Six-card grid item | White card, ink text | Black card, white text (full inversion, 250ms) |
| Stay-connected channel card | White card, 1px rule border | Black card, white text, lifted 2px, arrow shifts right 3px |
| Footer link | Light grey | White text + 1px red underline animating from left |
| Footer lockup | Static | Lifts 1px |

Focus styles for keyboard users mirror hover styles, plus a 1px King's red `outline` with 3px offset (already specified in §8 Accessibility).

### 4.6 Imagery

- The hero in the mockup uses a placeholder gradient. The shipped site needs a real photograph slot here — recommend a high-resolution image of either the Hardware Hack event (the existing site uses such an image) or a Strand campus shot.
- Image treatment: subtle dark gradient overlay at the bottom for the optional caption pill.
- No other photographic content on the homepage in v1.

## 5. Information Architecture

The page reorganises (does not remove) the existing site's six "topic blocks" plus the seeking-volunteers callout and the social/AGM links.

```
1. Header — lockup + 9-item nav + primary CTA
2. Hero — mission statement + image + meta-line
3. Hardware Hack 26 callout band — current high-priority CTA
4. "What the Association does" — six content cards
5. "Stay connected" — social channels + contact
6. Footer — brand + 3 link columns + copyright
```

### 5.1 Header

- Lockup (red square + "KCLEA / King's College London Engineers' Association")
- Nav (Home, Events, Bulletin, Annual Lunch, Annual Lecture, Committee, About Us, In Memoriam, Contact) — Webmaster moves to footer.
- Primary CTA: *"Join KCLEA"* — pill button, ink background, white text. Hover: King's red.

### 5.2 Hero

- Kicker (red, uppercase, .2em letter-spacing): "King's College London · Alumni Association"
- H1 (Cormorant 64px, italic red on the second line): *"The Alumni Association for **King's Engineers**, past & present."*
- Lede (Inter 14.5px): the full mission paragraph from the existing site, **verbatim**: *"KCLEA is the Alumni Association for those who have studied Engineering or work at King's in the Faculty or Division of Engineering — or more recently in the School of Biomedical Engineering and Imaging Science, or the Department of Engineering."*
- Meta-line below a 1px rule: the existing "Find out more on this website, on social media, or attend our AGM. Alumni keep up to date with KCLEA News — Join the KCLEA group on LinkedIn or on King's Connect."
- Image slot (right column): real photo recommended (Hardware Hack or Strand). Placeholder gradient in mockup.

### 5.3 Hardware Hack callout

- Black band, full width.
- Red Caslon-italic badge: "Now"
- H3 (white Cormorant 24px): *"KCLEA is seeking members to help with **Hardware Hack 26**"*
- Sub-line: "Volunteer to mentor students at the next King's engineering challenge."
- White pill CTA: "Get involved →"

### 5.4 What the Association does — six-card grid

3-column × 2-row grid, no shadows, 1px rule gaps. **No permanent "feature" card** — all six cards default to white and switch to the black inverted treatment on `:hover` / `:focus-within`.

| # | Card | Source |
|---|---|---|
| 01 | Membership | Membership block |
| 02 | Annual *Lunch* | Annual Lunch block |
| 03 | Newsletters & *Bulletin* | Newsletters & Bulletin block |
| 04 | Find out about King's Events | Find out about Kings Events block |
| 05 | Annual *Lecture* | Annual Lecture block |
| 06 | Activities *with Students* | Activities with Students block |

**Hover/focus interaction:**
- Default state — white background, ink-coloured headline, muted-ink body, red number/accent.
- Hover or keyboard-focus state — black (`#0d0d0d`) background, white headline, light-grey body (`#bcbcbc`), red accent retained.
- Transition — `background-color` and `color` 250ms ease.
- Cursor — `pointer` (each card is a clickable surface; if the card is purely text-with-inline-links, omit `pointer` and only the inner `<a>` elements are interactive — implementation detail to be decided when scaffolding).
- The `:focus-within` mirror is required so keyboard users get the same affordance.

Each card preserves:
- The original headline text (with light italicisation of the most evocative word)
- The original body paragraph
- All inline links (Membership Secretary, AGM on Zoom, Events Calendar, LinkedIn, online Bulletin, bulletin@kclea.org.uk, contact form, KCLES, KCL WiSTEM, BMEIS, Dept. of Engineering)

### 5.5 Stay connected

Two-column band on warm paper background.

Left:
- Kicker "Stay connected"
- H2: *"Follow KCLEA and the **King's network**"*
- Body: existing copy from the homepage's social/Connect sentences.
- Italic red hint: "It helps if you have your subject and graduation date in your profile." (verbatim from existing site)

Right: 2×2 channel grid:
- KCLEA Group on LinkedIn
- Follow KCLEA on LinkedIn
- King's Connect
- Get in touch (Contact)

### 5.6 Footer

Dark band. Four columns:
1. Lockup + tagline paragraph
2. The Association — Home, About Us, Committee, In Memoriam, Contact Us
3. Events & Bulletin — Events, Annual Lunch, Annual Lecture, Bulletin
4. Site — Accessibility, Privacy, Terms of Use, Site Map, Webmaster

Bottom row: "© 2018–2026 King's College London Engineers' Association. All rights reserved." + "Updated: 26-04-2026".

## 6. Components (each with one clear purpose)

| Component | Purpose | Inputs | Internal? |
|---|---|---|---|
| `KcleaHeader` | Wordmark, primary nav, primary CTA | active page slug | No external dependencies |
| `Hero` | Mission framing + entry CTA + image | headline, lede, meta-line links, image src+alt | Pure presentation |
| `HackCalloutBand` | Promote a single high-priority current ask | badge text, H3, sub, CTA href | Replaceable as season changes |
| `SixCardGrid` | Surface the six "what we do" topics | array of `{number, title, body, links, isFeature?}` | Item card is a sub-component |
| `StayConnected` | Promote social channels + contact | array of channels | Pure presentation |
| `KcleaFooter` | Wayfinding to all secondary pages, legal | nav columns, copyright string | No external dependencies |

Each component is self-contained: it accepts data in, renders presentation out, and depends only on the shared design tokens (font family CSS variables, colour variables, spacing scale).

## 7. Tech & Deliverable

**Confirmed: Static HTML/CSS, no build tools.** Matches the simplicity of the existing Squarezone hosting, no JavaScript framework, easy for a non-developer committee to maintain.

- Single `index.html` with one external `styles.css` (preferred over embedded `<style>` for easier inner-page reuse later)
- Google Fonts via `<link>` tag for Cormorant Garamond + Inter
- **No JavaScript required for v1** — no carousels, no expand/collapse, no menu toggle script (mobile menu uses native `<details>` if needed)
- Responsive via plain CSS grid + media queries (breakpoint at ~860px collapses 3-col grids to single column; below ~640px the nav becomes a `<details>`-based disclosure)
- Deploys as static files anywhere (Netlify, GitHub Pages, the current Squarezone if it accepts custom HTML, or any plain Apache/Nginx host)
- Local dev server during development: `python3 -m http.server 8080` from the project root — no install required

## 8. Accessibility

- Body copy ≥ 13px (Inter renders well at this size). Headlines and lede use larger sizes.
- Colour contrast: King's Red `#D3231A` against white = 4.94:1 (AA for normal text). Against black = 4.25:1 (AA for large text only — fine for headlines, NOT used for body copy on dark backgrounds).
- All interactive elements receive a visible `:focus-visible` outline (1px King's red, 3px offset).
- Semantic HTML: one `<h1>` per page (the hero), `<nav>` around the menu, `<main>` around content, `<footer>` for the footer. Anchor tags only on real links.
- Alt text on the hero image describing the scene (not "image of a Hardware Hack").

## 9. Out of scope (deferred)

- Inner pages (next sprint)
- A real CMS / dynamic content (the existing Squarezone setup may continue managing content, or a static-site rebuild can be specced separately)
- Member-only authenticated areas
- Event RSVP / payment flows
- Bulletin archive search
- Email newsletter integration
- Translations (English-only)

## 10. Open questions

Resolved during brainstorm:

- ~~**Final tech stack**~~ — Static HTML/CSS, no build tools. *Resolved.*
- ~~**Logo**~~ — A new KCLEA logo is in production but not yet ready. The homepage ships with the text-based red square lockup as a placeholder; swap path documented in §4.3. *Resolved.*
- ~~**Highlighted feature card**~~ — No fixed feature card; all six cards switch to black on hover/focus. *Resolved.*
- ~~**Footer secondary links**~~ — Accessibility, Privacy, Terms of Use, Site Map link to the same targets they do on the existing site. *Resolved.*

Still open (do not block planning, can be answered during implementation):

1. **Hero image:** which photograph(s) does the committee have rights to? Ship v1 with the placeholder gradient if no image is ready by build time.
2. **Hardware Hack 26 link target:** does this campaign already have a landing page, or does the CTA point to an email / contact form?
3. **Hosting / deployment:** keep Squarezone as host, or move to a static host (Netlify / GitHub Pages)?

## 11. Success criteria

- The homepage preserves every meaningful link and content paragraph from the current `index.html`.
- A first-time visitor can identify (a) what KCLEA is, (b) how to join, and (c) what's happening soon (Hardware Hack 26, Annual Lunch 13 Jun) — all above the second scroll.
- Visual style is recognisably "King's College London" by anyone familiar with the parent brand.
- No JavaScript dependency for the page to function.
- Lighthouse scores ≥ 95 across Performance, Accessibility, Best Practices, SEO.
