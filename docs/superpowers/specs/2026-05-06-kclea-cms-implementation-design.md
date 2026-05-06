# KCLEA CMS — Implementation Spec (one-pager)

**Date:** 2026-05-06
**Status:** Ready for build · estimated 5–7 working days with Claude Code
**Companion:** `2026-04-29-kclea-homepage-redesign-design.md`, `…events-redesign-design.md`

---

## 1. Goal

Give KCLEA committee members the ability to **edit the existing site's content** without touching code — text edits, event listings, bulletin uploads, roster updates. Layout, typography, components, and section structure are **locked** to the already-approved design.

## 2. Scope

**In:** edit text on every existing page; CRUD for Events; CRUD for Bulletin issues (PDF + cover image); CRUD for the 7 list collections below; upload PDFs and images; two user roles.

**Out:** new page types, new section types, content migration from the legacy site, payments, RSVPs, member directory with member login, audit log, draft/publish, scheduled publish, multi-language.

## 3. Stack

| Layer | Choice | Why |
|---|---|---|
| CMS / API | **Payload CMS** (self-hosted, MIT) | TypeScript schemas, built-in auth + RBAC + media |
| Database | **Postgres** | Standard, hosted on Railway / Render |
| File storage | **S3-compatible** (Cloudflare R2 or AWS S3) | PDFs + image variants |
| Frontend | The static site we already built, converted to **per-page templates** that fetch from Payload at build time | Keeps the live site fast and unchanged in look |
| Hosting | Payload + DB on **Railway**; static frontend on **Cloudflare Pages** | One-click deploy hooks; rebuild on Save |
| Build | **Static-site rebuild on every Save** (~30s) | No client-side JS dependency on the CMS |

## 4. Roles

| Role | Can | Cannot |
|---|---|---|
| **Editor** | Edit any content; upload PDFs & images; add/edit/delete entries in any list collection | Manage users; change site settings |
| **Super Admin** | Everything Editor can + create / disable / reset password for users | — |

Roles enforced via Payload's `access` functions on every collection.

## 5. Content models

### 5.1 Page collections (text-only; one document per page; fields locked to the design)

`HomePage`, `EventsPage`, `BulletinPage`, `AnnualLunchPage`, `AnnualLecturePage`,
`PastLecturesPage`, `CommitteePage`, `AboutPage`, `MembershipPage`, `BursariesPage`,
`ConstitutionPage`, `ThirteenClubPage`, `DirectoryPage`, `UsefulLinksPage`,
`InMemoriamPage`, `ContactPage`.

Each page document holds the typed text fields for its hero, sections, and CTA bands — grouped in the Payload admin so the Editor sees, e.g., **"Hero" / "Six cards" / "Stay connected"** as collapsible blocks rather than a flat list.

### 5.2 List collections (CRUD)

| Collection | Fields | Used on |
|---|---|---|
| `Events` | slug, title, kicker, dateStart, dateEnd, timeStart, timeEnd, venue, postcode, sessions[], pills[], heroImage, body (rich text), nextEventRef | Events index + auto-generated event-detail pages |
| `BulletinIssues` | season ("Spring 2026"), issueNumber, volumeNumber, coverImage, pdfFile, summary, publishedDate | Bulletin page (latest + archive) |
| `CommitteeMembers` | group (Executive / Officers / Members / Trustees), roleTitle, name, yearsAtKings, isVacancy, sortOrder | Committee page |
| `InMemoriamEntries` | name, prefix ("Professor" / "Dr"), birthYear, deathYear, sortOrder | In Memoriam page |
| `PastLectures` | year, title, speakerName, speakerTitle, isInaugural, isOnline, dateLine, venue, abstract (rich text), speakerBio (rich text), recordingUrl | Past Lectures page |
| `BursaryWinners` | year, name, course, type ("Bursary" / "Medal"), department | Bursaries page |
| `ThirteenClubWinners` | year, name, courseAndYears, citation, isFeatured, sortOrder | 13 Club Trophy page |
| `UsefulLinks` | label, description, url, sortOrder | Useful Links page |

### 5.3 Site settings (single global doc)

`primaryEmail`, `bulletinEmail`, `presidentEmail`, `lunchEmail`, `linkedInGroupUrl`, `linkedInPageUrl`, `kingsConnectUrl`, `copyrightYearStart`.

### 5.4 Users

`email`, `name`, `role` (`editor` | `super_admin`), Payload-managed password.

## 6. File upload rules

| Type | Constraints | Auto-generated variants |
|---|---|---|
| Hero / banner image | ≤ 5 MB · JPG/PNG/WebP | 600×174 (panoramic), 1600×900 (16:9), 1200×900 (4:3 mobile) |
| Cover image (Bulletin) | ≤ 3 MB · JPG/PNG | 480×720 (3:2 portrait), 240×360 thumb |
| Avatar / portrait | ≤ 2 MB · JPG/PNG | 200×200 square, 400×400 retina |
| PDF (Bulletin, Constitution, Accounts) | ≤ 25 MB | none — served as-is |

## 7. Frontend integration

- The current single-file SPA is split into **per-page Astro / Next-static templates**, one per slug.
- Each template fetches its `Page` document + relevant list collections at build time and renders the design exactly as approved.
- A Payload **afterChange hook** triggers a Cloudflare Pages deploy webhook on every Save → site rebuilds and is live in ~30 seconds.
- Asset paths point at `/assets/uploads/...` (Cloudflare R2 + CDN).

## 8. Acceptance criteria (proof of done)

1. Editor can log in, edit any text on any of the 16 pages, click Save, and see the change live within 60 seconds.
2. Editor can add a new event, upload a Bulletin issue (PDF + cover), add a committee member, and add an In Memoriam entry — each appears on the appropriate live page after Save.
3. Editor cannot create or delete users; the user-management screens are hidden.
4. Super Admin can create a new Editor account and reset passwords.
5. Image uploads generate the correct cropped variants automatically; PDF uploads are served from the CDN.
6. Lighthouse Accessibility score ≥ 95 preserved after CMS integration.

## 9. Day-by-day plan (5–7 working days)

| Day | Output |
|---|---|
| 1 | Payload + Postgres + R2 + auth + roles + deploy pipeline |
| 2 | All 16 page schemas + 8 list collections + site settings |
| 3 | File upload variants + Editor-admin field grouping & descriptions |
| 4 | Per-page template conversion + build webhook |
| 5 | Smoke test every model end-to-end + fix 2–3 small bugs + production deploy |
| 6–7 | Buffer for image-crop edge cases, hosting friction, or late field tweaks |

## 10. Out of scope (future sub-projects)

- Member-only RSVPs / payments
- Email newsletter integration (Mailchimp / Brevo)
- Public member directory with self-edit
- Multi-language
- Audit log + draft/publish workflow

— END —
