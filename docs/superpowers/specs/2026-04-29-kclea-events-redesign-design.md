# KCLEA Events — Index + Event Detail Template — Design Spec

**Date:** 2026-04-29
**Author:** brainstormed with Claude
**Sources:**
- Events index: https://www.kclea.org.uk/events.html
- Worked example detail page: https://www.kclea.org.uk/events/hardhacks26.html
- Combined preview: `.superpowers/brainstorm/4296-1777452322/kclea-site-v1.html`
**Companion spec:** `2026-04-29-kclea-homepage-redesign-design.md` — defines the brand language, header, footer, and design tokens shared across all pages. This spec inherits everything in §4 (Visual Language) of the homepage spec.

---

## 1. Context

The current Events page is a single small HTML table (3 rows) of upcoming dates with no detail beyond date / title / venue / time. Each event optionally links to its own detail page (`/events/<slug>.html`) — for example, the Hardware Hacks 26 page is a long-form, plain-text article without typographic hierarchy or distinct sections. The redesign keeps every word but reorganises into a magazine-style index and a structured detail template that can be reused across all events.

## 2. Goals

- **Same content, new structure.** No editorial cuts. All venue addresses, sessions, role descriptions, and travel directions preserved.
- **A reusable event-detail template** that adapts to different event types (hackathon, formal lunch, lecture). Sections that don't apply to a given event are simply omitted.
- **Index → detail wiring.** Each event row in the index links to its detail page; from any detail page the breadcrumb returns to the index, and a "Coming up next" footer links to the chronologically following event.
- **Inherit the homepage brand language** — same header, footer, fonts, colours, hover model.

## 3. Scope

**In scope:**
- The Events index page (`events.html`)
- The Event detail page template (one `.html` per event, e.g. `events/hardhacks26.html`)
- The Hardware Hacks 26 detail page as the first concrete instance using the template

**Out of scope (this spec):**
- Past Annual Lectures archive page (separate spec — linked from the Events index "Archives & news" block)
- News Items page (separate spec)
- Event RSVP / payment flows
- An actual calendar feed / iCal export
- Inner pages other than Events (Bulletin, Annual Lunch landing, Annual Lecture landing, Committee, About, In Memoriam, Contact)

## 4. Inheritance

This page uses, without redefining, the following from the homepage spec (`2026-04-29-kclea-homepage-redesign-design.md`):

- §4.1 Typography (Cormorant Garamond + Inter)
- §4.2 Colour tokens (`--kings-red`, `--ink`, `--paper`, `--paper-warm`, `--rule`, etc.)
- §4.3 Lockup (red square text placeholder, swap path documented)
- §4.4 Layout & Spacing rules
- §4.5 Interaction (hover & focus model)
- §5.1 Header
- §5.6 Footer

Everything below describes only what is **specific to the Events sub-system**.

---

## 5. Events Index Page (`events.html`)

```
Header  ─ shared
Hero    ─ title + lede + "this month" sidebar tile
Month   ─ heavy-rule month heading + rows
  Row × N  (date numerals · event · time · tag)
Disclaimer band  (warm paper, red bullet)
Archives & news  (4-card link grid)
Footer  ─ shared
```

### 5.1 Hero

- **Kicker** (red, .2em letter-spacing): `KCLEA · Calendar of Events`
- **H1** (Cormorant 64px, italic red on second word): *"Upcoming **events.**"*
- **Lede:** *"Forthcoming KCLEA events for King's engineering alumni and staff. Free to attend; all welcome unless otherwise noted. Scroll the calendar below — and check back regularly as new dates are added throughout the year."*
- **Sidebar tile** (right column, ~33% width, divided by 1px rule):
  - Label: `This month`
  - Big italic: `<N> events`
  - Meta line: short context (e.g. "In June 2026 — including the flagship Annual Lunch")

### 5.2 Month section

- Heavy 2px ink-coloured rule under the month heading.
- Heading: Cormorant 34px, italic red year (e.g. `June 2026`)
- Right-aligned count label (`Three events`)
- Each row repeated below — multiple month sections stack vertically when there are events in different months.

### 5.3 Event row (the unit)

Grid: `110px | 1fr | auto | auto`

| Column | Content | Type |
|---|---|---|
| 1 — Date | Big Cormorant numeral (64px), day name below in Inter | display |
| 2 — Event | Cormorant 24px headline (italic red on the campaign name), venue + post-code muted line under | content |
| 3 — Time | Italic red Cormorant, sub-line (e.g. "3 sessions") below in Inter | display |
| 4 — Tag | Pill — Volunteer (red on blush) / Flagship (white on ink) / etc. | classification |

**Hover/focus state:** the row gets a `--paper-warm` background and the date numeral switches to King's red. Padding shifts in 14px on each side to suggest "this row is moving forward." Whole row is the click target.

**Anchor:** the row's click target is `<a href="events/<slug>.html">` wrapping the row content (or the row is `<a class="e-row">` in v1). Each event in the index that has a detail page is linked. Events that have no dedicated detail page show no hover lift and are not links.

### 5.4 Disclaimer band

Warm-paper background. Red circular bullet (26px, exclamation glyph in Cormorant), italic ink-soft text:

> "The above dates may change without warning. Members are encouraged to check the email newsletter and KCLEA LinkedIn page closer to each event date."

(The original page's disclaimer is preserved verbatim and slightly extended with an actionable suffix.)

### 5.5 Archives & news (4-card grid)

Two-column on the left (kicker + Cormorant H2 + paragraph), 2×2 grid of link cards on the right:

1. **Past Annual Lectures** — links to the lectures archive
2. **News Items** — links to news index
3. **The Bulletin** — links to bulletin index
4. **Follow on LinkedIn** — external

Each card uses the same hover treatment as the homepage's "Stay connected" channel cards: white → black, lifts 2px, arrow shifts right.

---

## 6. Event Detail Template

The template is composed of optional sections in a fixed order. Each event populates whatever is relevant; unused sections are simply omitted.

```
Header           ─ shared
Breadcrumb       ─ Home / Events / <event title>
Hero             ─ title, standfirst, info pills, CTAs + at-a-glance fact card
─── About the event           [optional, almost always used]
─── How it works (4-step)     [optional — used for activity-based events]
─── Pull quote (red band)     [optional — for distinctive event slogans]
─── Roles                      [optional — for events seeking volunteers]
─── CTA band (dark)            [optional — for events with a clear ask]
─── Map strip                  [optional — for events with a venue]
─── Location & travel (3-card) [optional — for events with a venue]
─── Stay updated (mini)        [optional — short LinkedIn / King's Connect prompt]
─── Coming up next             [always — link to next event chronologically]
Footer           ─ shared
```

### 6.1 Breadcrumb

Warm-paper strip. `Home / Events / <current event>`. The trailing item is bold ink. Earlier items are King's red on hover-out, ink on hover. Used as the primary "back" affordance.

### 6.2 Hero

Two-column grid (1.4fr / 1fr).

**Left column:**
- Kicker — `KCLEA Event · <date range>`
- H1 — Cormorant 60px, with one italic red word (typically the event number/year)
- **Standfirst** — Cormorant *italic* 21px, ink-muted. One sentence summary, typographically distinguished from body copy.
- **Info pills** (4 max) — date range, time, venue, optional campus/region. The venue pill uses the inverted dark variant.
- **CTAs** — pill buttons. Primary is King's red filled; ghost is bordered ink.

**Right column — "At a glance" fact card:**
- Warm-paper background, 1px rule border, 6px radius
- Heading "At a glance" with italic red accent
- 4–6 fact rows, each `90px lab | 1fr value`. Labels: When / Where / Hosts / Roles / Contact / Cost / Format. Lab is uppercase 10px, .18em letter-spacing.

### 6.3 About the event

White section, single H2, one or two paragraphs of body copy in `--ink-muted` at 15px. Inline links use the homepage's red faint-underline style with red wash on hover.

### 6.4 How it works (4-step grid)

Used when the event has a process (a hackathon, a workshop, etc.). Section background is `--paper-warm`.

- 4-column grid, 1px rule between cells
- Each step: italic red Cormorant numeral (32px), Cormorant 20px headline, small body copy
- Hover inverts the step to black + white text (mirrors the homepage 6-card pattern)
- Number stays King's red on hover

### 6.5 Pull quote band

Used for memorable event slogans or rules. Full-width King's red band, white Cormorant *italic* 32px serif, centred, max-width 820px. May contain an Inter uppercase strong line for emphasis (e.g. "IF IT DOESN'T PLUG IN, MOVE, SENSE, OR INTERACT WITH THE PHYSICAL WORLD — IT DOESN'T QUALIFY").

### 6.6 Roles

Used when the event needs volunteers, judges, mentors, etc. White section, two-column grid (1fr/1fr), 1px rule between cells.

Each role card:
- Role tag — red 10px uppercase
- H3 with italic red name (e.g. "*Supporters*")
- Description paragraph
- Optional sessions list — `130px when | 1fr what` rows, italic Cormorant for the time, Inter body for the description
- Role meta — italic muted footer line for credits (e.g. "Dr Mike Clode will chair the team of 5 judges")

### 6.7 CTA band

Black band, two-column grid. Left: kicker + Cormorant H2 (italic red accent) + ink-muted body. Right: stacked CTA stack — primary email shown as italic Cormorant red text with red faint underline, then a primary red pill button, then a ghost button.

### 6.8 Map strip

Slim 160px-tall strip with a diagonal-stripe placeholder pattern and a King's-red pill marker labelled with the venue. **Implementation note:** v1 uses a CSS-pattern placeholder. v2 (post-launch) can swap in an embedded OpenStreetMap or static map image — the strip's dimensions stay fixed.

### 6.9 Location & travel

White section, three-column 1px-rule grid:

1. **Address card** — venue name in Cormorant H4, full address, optional small tag chip ("5 min from London Bridge")
2. **From the Tube card** — directions paragraph
3. **From the mainline card** — directions paragraph

Some events may need only one directions card; the grid collapses to 2 columns or a single card row.

### 6.10 Stay updated (mini)

Warm-paper section. H2 with italic red ("*Stay updated.*") and a single 15px paragraph linking to the KCLEA LinkedIn group, KCLEA LinkedIn page, and King's Connect. Echoes the homepage's "Stay connected" section but compressed to one paragraph.

### 6.11 Coming up next

White section. Heavy 2px ink rule above. Single row:

- Left: kicker + Cormorant H3 (italic red on event name) + venue muted line
- Centre: big Cormorant date numeral + "Sat · Jun" sub-label
- Right: italic Cormorant red arrow

Hover shifts padding inwards and background to warm paper. The whole row is the link target. Computes the *next chronological event after the current one* — if the current event is the last one in the calendar, this section is replaced by a "Browse all events →" affordance.

---

## 7. Components

| Component | Purpose | Inputs |
|---|---|---|
| `EventsIndexHero` | Page lede + this-month sidebar | `kicker, headline, lede, monthLabel, eventCount, monthMeta` |
| `MonthSection` | One month's worth of events | `monthLabel, year, events[]` |
| `EventRow` | A single event in the index | `slug, day, dayName, title, titleEm, venue, postcode, sessionsLine, timeStart, timeEnd, timeSub, tag` |
| `Disclaimer` | The "dates may change" band | `text` |
| `ArchivesGrid` | 4-card "beyond this month" grid | `cards[]` |
| `Breadcrumb` | Trail at top of any inner page | `items[]` (last is current) |
| `EventHero` | Detail-page hero | `kicker, headline, headlineEm, standfirst, pills[], ctas[]` |
| `FactCard` | At-a-glance side panel | `heading, rows[{lab, val}]` |
| `Prose` | Body copy section | `heading, body[]` |
| `StepsGrid` | Four-step process grid | `heading, steps[{n, title, body}]` |
| `PullQuoteBand` | Red quote band | `body, strongLine?` |
| `RolesGrid` | 1- or 2-role grid | `heading, intro, roles[{tag, title, body, sessions[], meta}]` |
| `CtaBand` | Black-band ask | `kicker, headline, body, primaryEmail, primaryButton, ghostButton` |
| `MapStrip` | Placeholder map | `pinLabel` |
| `TravelGrid` | 3-card directions | `cards[]` |
| `StayUpdated` | One-paragraph LinkedIn prompt | `body` |
| `NextUpRow` | Chronological next-event link | `slug, kicker, title, titleEm, venue, day, monthShort` |

Each component is a pure presentation unit — accepts data, renders HTML. No JavaScript dependencies.

## 8. Data shape (event manifest)

For a static-HTML implementation, each event is a hand-edited `.html` file. To make it possible to render the index from a single source of truth, define a tiny JSON manifest at `events/_events.json`:

```json
{
  "events": [
    {
      "slug": "hardhacks26",
      "title": "Hardware Hacks 26 — supporters",
      "titleEm": "Hardware Hacks 26",
      "headlineLine": "Members needed for",
      "date": "2026-06-06",
      "endDate": "2026-06-07",
      "dayLabel": "06",
      "dayName": "Saturday",
      "timeStart": "10:00",
      "timeEnd": "18:00",
      "timeSub": "3 sessions",
      "venue": "GCR Hodgkin Building",
      "postcode": "SE1 1UL",
      "sessionsLine": "Three sessions throughout the day",
      "tag": { "label": "Volunteer", "variant": "default" },
      "detailHref": "events/hardhacks26.html"
    }
  ]
}
```

V1 may simply hand-write the index rows in `events.html`. V2 (when more than ~6 events) can refactor to read from `_events.json` and render server-side or via a tiny build step.

## 9. Routing & file structure

```
/index.html
/events.html
/events/
  hardhacks26.html
  annual-lunch-2026.html
  <future events>.html
  _events.json     (optional manifest for v2)
/styles.css
/assets/
  fonts/  (or use Google Fonts <link>)
  images/
```

All cross-page links are plain `<a href="…">` in production. The brainstorm preview's JS-based `data-page-link` router is **preview-only** and is not part of the shipped site.

## 10. Accessibility

Same rules as the homepage spec §8, plus:

- Each `EventRow` is wrapped in a single `<a>` so the entire row is a hit target (keyboard and screen-reader friendly). Inner text is the link's accessible name.
- The detail page's hero pills are non-interactive `<span>` elements — they're labels, not links.
- The fact card uses a `<dl>` (`<dt>` for labels, `<dd>` for values).
- Steps grid uses `<ol>` with custom-styled markers via the `.step-num`. The numerical sequence is meaningful, so an ordered list is correct.
- Roles grid uses `<section>` per role with `<h3>`. Sessions list uses `<ol>` (chronological order matters).
- The `<title>` of each detail page is `"<Event title> — KCLEA"`.
- Each detail page sets `<meta name="description">` from the standfirst sentence.

## 11. Tech & deliverable

Inherits §7 of the homepage spec — static HTML/CSS, no build tools, Google Fonts via `<link>`, no JavaScript required.

The only addition: an optional tiny script (~30 lines) that, on the events index, reads `_events.json` and renders the rows client-side. **This is opt-in and not part of v1.** V1 hand-writes the index rows.

## 12. Out of scope

- Member-only "RSVP" forms
- Calendar/iCal export
- Past events archive (lives behind the "Past Annual Lectures" link)
- News Items page
- A general-purpose CMS

## 13. Open questions

Inherited from the homepage spec (logo, hero photo, hosting). Plus:

1. **Annual Lunch detail page content:** the existing `events/annual-lunch.html` (or wherever it lives) needs to be scraped before the second event detail is built. Out of scope for this spec — handled when we build that page.
2. **`Volunteer` CTA target:** for events seeking volunteers, does the primary CTA link to a contact form, an email `mailto:` link, or a dedicated volunteer page? V1 uses both: the email is shown as an italic Cormorant address, and the pill button links to the Contact page.

## 14. Success criteria

- The Events index preserves every line from the existing `events.html` (the three June 2026 rows + the disclaimer).
- The Hardware Hacks 26 detail page preserves every paragraph, every direction sentence, every role description, and every contact (`president@kclea.org.uk`, KCL Robotics, KCL Electronics, Dr Mike Clode credit).
- A first-time visitor on the Events index can identify what's coming up, click into any event, read the full detail, and find their way back via breadcrumb — without ever opening a JS console.
- The same template renders sensibly with whatever subset of sections a future event populates.
- Lighthouse scores ≥ 95 across Performance, Accessibility, Best Practices, SEO.
