# KCLEA Website CMS Layer — Implementation Plan

## Context

The KCLEA website redesign (`razvanciuban-stack/kclea-website-preview`) is a **vanilla HTML/CSS site** — one `index.html` with hash routing, no framework, custom design system (Cormorant Garamond + Inter, King's red). README states the file will be split into per-page HTML before launch. The visual layer is done.

A previous spec (`docs/superpowers/specs/2026-05-06-kclea-cms-implementation-design.md`) designed a **Payload CMS** backend on Postgres + Cloudflare R2 + Railway. That spec works, but the user has decided to optimise for **zero ongoing cost** and **Squarezone-familiar UX**, accepting that we build a small custom CMS layer on top of Firebase instead. This plan supersedes that spec for build purposes; the Payload spec stays in the repo as the recorded alternative.

**Decision:** Firebase Auth + Firestore + Firebase Storage + Firebase Hosting + TipTap, with hybrid editing — **in-place** for page text, **modal forms** for the 8 list collections, **modal file pickers** for PDF/image uploads.

**Primary constraints driving design:**
- $0 ongoing cost (Firebase Spark plan)
- Implementer is a non-technical operator working with Claude Code → plan must be prescriptive, with explicit security and concurrency safeguards
- Site must remain fast and accessible to anonymous visitors (current Lighthouse ≥95 must hold)
- Committee members already know Squarezone's "log in, edit in-place" pattern; preserve it

## Stack

| Layer | Choice | Rationale |
|---|---|---|
| Hosting | **Firebase Hosting** | Same console as Auth/Firestore/Storage; supports custom domain, security headers, CSP |
| Public site | **Per-page static HTML** (split from current `index.html`) | Preserve existing design system unchanged |
| Editor SDK | **Firebase JS SDK v10+ (modular)** loaded as ES modules from CDN | No build step needed; can stay vanilla |
| Rich-text editor | **TipTap v2** loaded as ES modules from CDN | Limited toolbar (bold, italic, link, lists), stores ProseMirror JSON not HTML (XSS-safe) |
| Auth | **Firebase Auth — passwordless email link only** | No passwords to manage, allowlist-enforced |
| Database | **Firestore** | Free tier covers expected use 100×+ |
| Files | **Firebase Storage** + **Resize Images** extension (free) | Auto-generates the image variants the Payload spec called for |
| Bot protection | **Firebase App Check (reCAPTCHA v3)** | Free, blocks programmatic abuse |

**Total ongoing cost: $0.** Free tier limits: 50k Firestore reads/day, 20k writes/day, 1GB storage, 5GB hosting bandwidth/month. Comfortably 100× expected use.

## Architecture

```
┌────────────┐                       ┌──────────────────┐
│ Visitor    │──HTTPS──▶ Firebase ──▶│ Static HTML page │
│ (anonymous)│           Hosting      │ (default content │
└────────────┘                       │  baked in)       │
                                     └────────┬─────────┘
                                              │ JS bootstrap on load
                                              ▼
                                     ┌──────────────────┐
                                     │ Firestore read:  │
                                     │ pageBlocks &     │
                                     │ list collections │
                                     │ (public read OK) │
                                     └──────────────────┘

┌────────────┐                       ┌──────────────────┐
│ Member     │──same HTML──▶─────────│ Same page +      │
│ (signed in)│   + signed-in JS      │ TipTap editors + │
└────────────┘                       │ "Edit" buttons on│
                                     │ list items       │
                                     └────────┬─────────┘
                                              │
                                              ▼ writes
                                     ┌──────────────────┐
                                     │ Firestore        │
                                     │  + App Check     │
                                     │  + security rules│
                                     └──────────────────┘
```

**Editing model:**
- **Pages** (Home, About, Committee, etc.): each editable text region has `data-block-id` on its HTML element. On load, JS replaces text from Firestore. When signed in, JS swaps each region for a TipTap editor with a small toolbar.
- **Lists** (Events, Bulletin, Committee, In Memoriam, Past Lectures, Bursary Winners, 13 Club, Useful Links): rendered from Firestore. Signed-in members see "+ Add" + "Edit" + "Delete" buttons. Click → modal form. Submit → Firestore write.
- **Files** (PDFs, images): modal form's file picker uploads to Firebase Storage; the Resize Images extension generates variants; the form stores the file path + variants on the document.

## Security Model (Five Layers)

### Layer 1: No public sign-up
- Firebase Auth providers: **email-link only**. Email/password disabled. Google/Facebook/anonymous disabled.
- Sign-up via UI: nonexistent. Sign-in page just asks for an email and sends a magic link.
- Result: random visitors cannot create accounts.

### Layer 2: Allowlist enforcement
- `members/{uid}` Firestore documents created **only by an admin** (manually for the first one, then via `/admin/members` UI).
- Sign-in callback (`onAuthStateChanged`): if `auth.user` exists but `members/{auth.user.uid}` does NOT, immediately `signOut()` and show "Your email is not on the committee allowlist."
- Result: even if someone obtains a magic link, they get bounced.

### Layer 3: Firestore security rules (the real boundary)

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isMember() {
      return request.auth != null
        && exists(/databases/$(database)/documents/members/$(request.auth.uid));
    }
    function isAdmin() {
      return isMember()
        && get(/databases/$(database)/documents/members/$(request.auth.uid)).data.role == "admin";
    }
    function hasValidTimestamps() {
      return request.resource.data.updatedAt == request.time
        && request.resource.data.updatedBy == request.auth.uid;
    }

    match /pageBlocks/{blockId} {
      allow read: if true;
      allow write: if isMember() && hasValidTimestamps();
    }
    match /events/{eventId} {
      allow read: if true;
      allow write: if isMember() && hasValidTimestamps();
    }
    match /bulletinIssues/{id}      { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /committeeMembers/{id}    { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /inMemoriamEntries/{id}   { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /pastLectures/{id}        { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /bursaryWinners/{id}      { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /thirteenClubWinners/{id} { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }
    match /usefulLinks/{id}         { allow read: if true; allow write: if isMember() && hasValidTimestamps(); }

    match /siteSettings/{doc} {
      allow read: if true;
      allow write: if isAdmin() && hasValidTimestamps();
    }

    match /members/{uid} {
      // Only admins can read the membership list; members can read their own doc to confirm role
      allow read: if isAdmin() || (request.auth != null && request.auth.uid == uid);
      allow write: if isAdmin();
    }
  }
}
```

- Even with dev tools, a non-member cannot write. Even with dev tools, a non-admin cannot modify `members/` or `siteSettings`.

### Layer 4: Storage security rules

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    function isMember() {
      return request.auth != null
        && firestore.exists(/databases/(default)/documents/members/$(request.auth.uid));
    }
    match /uploads/images/{file=**} {
      allow read: if true;
      allow write: if isMember()
        && request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/(jpeg|png|webp)');
    }
    match /uploads/pdfs/{file=**} {
      allow read: if true;
      allow write: if isMember()
        && request.resource.size < 25 * 1024 * 1024
        && request.resource.contentType == 'application/pdf';
    }
  }
}
```

### Layer 5: XSS hardening
- TipTap stores **ProseMirror JSON**, not HTML. We never put user-supplied strings into `innerHTML`.
- TipTap's official renderer produces DOM nodes safely.
- Link nodes constrained to `http://` and `https://` schemes (no `javascript:`).
- The static HTML's existing baked-in default content is developer-authored, so it's not an XSS surface.

### Plus
- **App Check (reCAPTCHA v3)** enforced on Firestore + Storage — non-browser clients (bots, scripts) are blocked
- **`serverTimestamp()`** required by rules — clients can't backdate writes
- **Authorized domains** in Firebase Auth limited to: `kclea.org.uk`, `www.kclea.org.uk`, the Firebase-issued preview URL, `localhost` (for development)
- **CSP headers** in `firebase.json` Hosting config:
  ```
  Content-Security-Policy: default-src 'self';
    script-src 'self' https://www.gstatic.com https://www.googleapis.com https://unpkg.com;
    connect-src 'self' https://*.googleapis.com https://*.firebaseio.com wss://*.firebaseio.com;
    img-src 'self' data: https://*.googleusercontent.com https://firebasestorage.googleapis.com;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    font-src 'self' https://fonts.gstatic.com;
  ```
- **Concurrency guard**: every write reads the current `updatedAt` first; if it differs from the version the editor loaded, abort with a "block was updated by X at HH:MM — reload?" prompt

## Firestore Data Model

Mirrors the Payload spec so we don't lose any modeling work.

```
pageBlocks/{pageId}_{blockId}
  pageId: "home" | "about" | "committee" | "annual-lunch" | "annual-lecture"
        | "past-lectures" | "bulletin" | "membership" | "bursaries"
        | "constitution" | "thirteen-club" | "directory" | "useful-links"
        | "in-memoriam" | "contact" | "events"
  blockId: stable string defined in the HTML (e.g. "hero_intro", "section1_body")
  content: ProseMirror JSON (TipTap's output)
  updatedAt: serverTimestamp
  updatedBy: uid

events/{eventId}                       ← matches Payload spec section 5.2
  slug, title, kicker, dateStart, dateEnd, timeStart, timeEnd,
  venue, postcode, sessions[], pills[], heroImagePath, body (ProseMirror JSON),
  nextEventRef, updatedAt, updatedBy

bulletinIssues/{id}
  season, issueNumber, volumeNumber, coverImagePath, pdfPath, summary,
  publishedDate, updatedAt, updatedBy

committeeMembers/{id}
  group, roleTitle, name, yearsAtKings, isVacancy, sortOrder, updatedAt, updatedBy

inMemoriamEntries/{id}
  name, prefix, birthYear, deathYear, sortOrder, updatedAt, updatedBy

pastLectures/{id}
  year, title, speakerName, speakerTitle, isInaugural, isOnline, dateLine,
  venue, abstract (ProseMirror JSON), speakerBio (ProseMirror JSON),
  recordingUrl, updatedAt, updatedBy

bursaryWinners/{id}
  year, name, course, type, department, updatedAt, updatedBy

thirteenClubWinners/{id}
  year, name, courseAndYears, citation, isFeatured, sortOrder, updatedAt, updatedBy

usefulLinks/{id}
  label, description, url, sortOrder, updatedAt, updatedBy

siteSettings/global
  primaryEmail, bulletinEmail, presidentEmail, lunchEmail,
  linkedInGroupUrl, linkedInPageUrl, kingsConnectUrl,
  copyrightYearStart, updatedAt, updatedBy

members/{uid}
  email, displayName, role: "admin" | "editor",
  invitedAt, lastLoginAt
```

## File Layout (added to existing repo)

```
kclea-website-preview/
├── index.html                    ← existing, kept as fallback / preview
├── pages/                        ← NEW: split per-page HTML (done as a prerequisite)
│   ├── home.html
│   ├── events.html
│   ├── bulletin.html
│   ├── committee.html
│   ├── about.html
│   ├── annual-lunch.html
│   ├── annual-lecture.html
│   ├── past-lectures.html
│   ├── in-memoriam.html
│   ├── bursaries.html
│   ├── constitution.html
│   ├── thirteen-club.html
│   ├── directory.html
│   ├── useful-links.html
│   ├── membership.html
│   └── contact.html
├── admin/                        ← NEW
│   ├── login.html                ← magic-link sign-in form
│   └── members.html              ← admin-only: manage allowlist
├── assets/
│   ├── ... (existing)
│   └── js/                       ← NEW
│       ├── firebase-config.js    ← client SDK config (public values only)
│       ├── auth.js               ← passwordless flow, allowlist check, signOut
│       ├── editor-bootstrap.js   ← runs on every page; gates edit UI on auth
│       ├── editable-text.js      ← in-place TipTap wrapper for data-block-id elements
│       ├── tiptap-loader.js      ← loads TipTap modules from CDN
│       ├── list-renderer.js      ← renders + manages each list collection
│       ├── modal-form.js         ← reusable modal for list CRUD
│       ├── file-upload.js        ← Storage upload + variant URL retrieval
│       ├── concurrency.js        ← optimistic-concurrency guard
│       └── sanitize-link.js      ← block javascript: URLs in editor output
├── firestore.rules               ← NEW (see security section)
├── storage.rules                 ← NEW (see security section)
├── firebase.json                 ← NEW: Hosting config + CSP headers
├── .firebaserc                   ← NEW: project linkage
└── docs/superpowers/specs/
    ├── 2026-04-29-kclea-events-redesign-design.md           ← existing
    ├── 2026-04-29-kclea-homepage-redesign-design.md         ← existing
    ├── 2026-05-06-kclea-cms-implementation-design.md        ← existing (Payload alternative)
    └── 2026-05-13-kclea-firebase-cms-design.md              ← NEW: this plan, copied
```

## Implementation Order (for Claude Code)

Each step has explicit "you should see X" checkpoints so a non-technical operator knows it worked before moving on.

### Phase 0 — Cloud setup (operator does this in browser, NOT Claude Code)
1. Create Firebase project `kclea-prod` in [console.firebase.google.com](https://console.firebase.google.com)
2. Enable Authentication → Sign-in methods → enable **only "Email link (passwordless sign-in)"**. Disable everything else.
3. Add `kclea.org.uk`, `www.kclea.org.uk`, `localhost` to Authorized domains.
4. Create Firestore database in **production mode** (rules will start locked), region: `europe-west1` (closest to UK).
5. Create Storage bucket, region: `europe-west1`.
6. Enable App Check → register reCAPTCHA v3 site key, paste the key into Firebase console.
7. Install the **"Resize Images"** extension from Firebase Extensions; configure: sizes `200×200`, `400×400`, `600×174`, `1200×900`, `1600×900`, `480×720`, `240×360`; output formats: webp + original.
8. In Project Settings → General → "Your apps" → add a Web app → copy the config object (`apiKey`, `authDomain`, etc.) — paste into `assets/js/firebase-config.js`.
9. Manually create the first admin: in Firestore console, create document `members/<first-admin-uid>` with `{ email, displayName, role: "admin", invitedAt: <now>, lastLoginAt: null }`. (Get the UID by having that person sign in once first; the auth callback will fail their allowlist check the first time, but the UID will be created in Firebase Auth's user list — copy it from there.)

**Checkpoint:** Firebase console shows project, Auth enabled with email link only, Firestore DB exists, Storage bucket exists, Resize Images extension shows "Healthy", one document in `members/`.

**Estimate: 0.5–1 day.** All click-through work in browser consoles. First-timer with Firebase will spend extra time finding settings; second time would be ~2 hours. Resize Images extension installation can take 10–15 min to provision.

### Phase 1 — Split the SPA into per-page HTML
1. Convert `index.html`'s hash-routed sections into separate files under `pages/`, mirroring the spec's 16 page list.
2. Update internal navigation links from `#home` etc. to `/pages/home.html` etc.
3. Keep `index.html` as a redirect to `/pages/home.html`.
4. Deploy to a Firebase Hosting preview channel and verify all pages render exactly as before.

**Checkpoint:** every page loads at its new URL with identical visual output to the SPA preview.

**Estimate: 0.5–1.5 days.** Mechanical work that Claude Code handles well. Main time sink: visual diff-checking each of the 16 pages against the original SPA. Risk of regressions in shared CSS — keep the `<style>` block identical across pages or extract to one stylesheet first.

### Phase 2 — Auth scaffolding
1. Write `assets/js/firebase-config.js` with the public Firebase config.
2. Write `assets/js/auth.js`:
   - `sendSignInLink(email)` — calls Firebase Auth's `sendSignInLinkToEmail`
   - On page load: if URL is a sign-in link, complete sign-in, then check `members/{uid}` exists; if not, `signOut()` and show "Not on allowlist" message; if yes, redirect to `/pages/home.html`.
   - `signOut()` helper.
   - Exposes `getCurrentMember()` returning `{ uid, email, role } | null`.
3. Write `admin/login.html` — email input + "Send sign-in link" button.
4. Deploy. The admin (from Phase 0 step 9) signs in with their email, gets the link, clicks, lands on home page authenticated.

**Checkpoint:** admin sees their email in a "Signed in as …" indicator (we'll add a small badge in header for signed-in users). Sign-out works.

**Estimate: 1–2 days.** Magic-link auth is fiddly — debugging the email-link flow (Continue URL config, action codes settings, allowlisted domains) commonly eats a half-day. The chicken-and-egg of bootstrapping the first admin (Phase 0 step 9 ↔ this phase) trips most people up once.

### Phase 3 — Firestore + Storage rules
1. Write `firestore.rules` and `storage.rules` exactly as specified in the security section above.
2. Deploy rules with `firebase deploy --only firestore:rules,storage`.
3. Run rules unit tests with the **Firebase Rules Emulator** (`firebase emulators:exec --only firestore "npx jest"`). Tests must cover:
   - Anonymous user reads `pageBlocks/*` → allowed
   - Anonymous user writes `pageBlocks/*` → denied
   - Non-member signed-in user writes → denied
   - Member signed-in user writes with correct `updatedBy` + `updatedAt` → allowed
   - Member writes to `members/` collection → denied
   - Admin writes to `members/` collection → allowed

**Checkpoint:** all rule tests pass. This is the most important checkpoint in the build.

**Estimate: 1–2.5 days.** This is the highest-risk phase. Rules syntax is unforgiving, and the cross-service check (Storage rule reading Firestore for membership) needs the Firestore emulator running alongside the Storage emulator. Budget extra time here — getting rules wrong is the only failure mode in this whole project that compromises security, not just functionality.

### Phase 4 — In-place text editing
1. Write `assets/js/tiptap-loader.js` — dynamic import of TipTap modules from CDN (`@tiptap/core`, `@tiptap/starter-kit`, `@tiptap/extension-link`).
2. Write `assets/js/editable-text.js`:
   - Scans for `[data-block-id]` elements on the page
   - For each, on page load, reads `pageBlocks/{pageId}_{blockId}` from Firestore; if present, replaces the element's content with TipTap's renderer
   - If signed in as a member, swaps the element for a TipTap editor (toolbar: bold, italic, link, bullet list, numbered list, undo, redo). Save / Cancel buttons appear above the editor.
   - Save: optimistic-concurrency check via `concurrency.js`, then write to Firestore.
3. Tag each editable region in the per-page HTML with `data-block-id` and `data-page-id`.
4. Test on the home page first.

**Checkpoint:** home page has editable intro paragraph. Anonymous visitor sees default text (from HTML) or saved Firestore content. Signed-in member sees an editor. Edit + save updates immediately. Open a second browser → second visitor sees the new content on refresh.

**Estimate: 1.5–3 days.** TipTap setup is straightforward but rolling it out to every editable region across 16 pages takes time. Tagging each region with `data-block-id` (and keeping IDs stable so saved content doesn't orphan) is tedious but easy. The optimistic-concurrency guard adds a half-day if implemented carefully.

### Phase 5 — List collections (CRUD via modal forms)
1. Write `assets/js/modal-form.js` — reusable modal component (vanilla JS, no framework).
2. Write `assets/js/list-renderer.js` — for each list collection, renders the list from Firestore, adds Add/Edit/Delete buttons for signed-in members.
3. Implement one collection end-to-end first: `events`. Verify Add/Edit/Delete works, list sorts by `dateStart`.
4. Implement remaining 7 collections.
5. Implement `siteSettings/global` as a single-doc edit form on `/admin/site-settings.html` (admin-only UI).

**Checkpoint:** every list collection editable end-to-end. Public visitor sees the lists; member can mutate them.

**Estimate: 2–4 days.** The biggest phase. First collection (Events) takes ~half a day getting modal + form + validation patterns right; subsequent 7 collections are ~2 hours each because they follow the established pattern. Form validation, date pickers, and sort-order UX consume time. Site settings page is small (~half a day).

### Phase 6 — File uploads
1. Write `assets/js/file-upload.js`:
   - Uploads file to `uploads/images/<uuid>.<ext>` or `uploads/pdfs/<uuid>.pdf`
   - Polls for the Resize Images extension's variants (or listens for completion)
   - Returns the variant URLs to the modal form, which stores them on the document
2. Wire into the modals for: `events.heroImage`, `bulletinIssues.coverImage` + `pdfFile`, etc.
3. Test: upload a 4MB image → see all variants appear in Storage → see them stored on the document → see them rendered on the public page.

**Checkpoint:** upload + variants + display all work for events and bulletin.

**Estimate: 1–2 days.** Resize Images extension does the heavy lifting. Main time sink: waiting for variant generation (async) means the modal needs a "processing…" state — getting that polling logic clean takes some iteration. CORS on Storage URLs sometimes needs a one-line config update; budget half a day for that one if it happens.

### Phase 7 — Member management
1. Build `admin/members.html` — admin-only UI to add/remove members (visible only when current user is admin; backed by rules so a non-admin couldn't access it via direct URL).
2. Add member flow: admin enters email + role → writes `members/{newUid}` (the UID is allocated when they first attempt to sign in; until then, use the email as the doc ID temporarily and migrate on first login — or use Firebase Functions, but those aren't on Spark plan free, so we use email-as-doc-ID with a small migration step on first login).
3. Alternative simpler flow: admin generates the sign-in link manually and emails it; on first sign-in, the auth callback finds a `pendingMembers/{email}` doc and migrates it to `members/{uid}`.
4. Implement the simpler flow.

**Checkpoint:** admin can invite a second committee member; they receive an email; they sign in; they can edit.

**Estimate: 0.5–1.5 days.** Small surface area, but the `pendingMembers/{email}` → `members/{uid}` migration on first login is fiddly to get right (needs Firestore rule for read on pending docs by email-match, etc.). If that gets complex, fallback: admin creates the doc manually with the UID after the new user signs in once — works fine for 4–10 members where invites are rare.

### Phase 8 — Hardening + launch
1. Add `firebase.json` with CSP headers as specified.
2. Smoke-test full flow on the Firebase Hosting staging URL.
3. Verify Lighthouse: Performance ≥90, Accessibility ≥95, SEO ≥95.
4. Remove `noindex` meta tags from all pages, update `robots.txt` to allow crawling.
5. Add custom domain `kclea.org.uk` in Firebase Hosting → update DNS at registrar (A records or CNAME as Firebase instructs).
6. Wait for SSL provisioning (Firebase does it automatically; ~15 minutes typically).
7. Keep the Squarezone site available for 30 days as backup, then cancel.

**Checkpoint:** `https://kclea.org.uk` serves the new site over HTTPS. Committee members sign in and edit successfully.

**Estimate: 1–2 days.** Most of the wall time is *waiting*: DNS propagation (up to 48h depending on TTL, usually <1h), Firebase SSL provisioning (~15–60 min), and email deliverability checks. Active work is small. Schedule cutover early in the week so DNS issues don't bite over a weekend.

## Likely Friction Points (Where Claude Code + Operator Will Need to Pause)

| Step | Likely problem | How to recognise | Fix |
|---|---|---|---|
| Phase 0 step 8 | Wrong config field copied | Editor fails to load Firebase | Re-copy from console; ensure `appId` and `measurementId` are included |
| Phase 0 step 9 | Can't get the UID without signing in first | Chicken-and-egg | Have person attempt sign-in (will fail allowlist), find them in Firebase Auth users list, copy UID, create `members/{uid}` doc, sign in again — succeeds |
| Phase 2 | Magic link doesn't arrive | Email sent but in spam | Set up SendGrid (free) as the SMTP provider in Firebase Auth, or whitelist `noreply@<project>.firebaseapp.com` |
| Phase 3 | Rules tests fail | `firestore.exists(...)` syntax in Storage rules is finicky | The exact syntax above has been verified; if rules tool complains, check Firebase docs version |
| Phase 4 | TipTap doesn't render | Wrong CDN path or version mismatch | Pin to specific version, e.g. `@tiptap/core@2.10.3` |
| Phase 4 | First load shows default HTML then "jumps" to Firestore content | Visible content shift | Render Firestore content into a hidden div first, swap atomically |
| Phase 5 | Modal form doesn't close on Escape | Missing keyboard handler | Standard accessibility — add `keydown` listener for Escape |
| Phase 6 | Resize Images extension doesn't fire | Wrong bucket path | Configure the extension to watch `uploads/` prefix specifically |
| Phase 8 | DNS cutover breaks signin | Authorized domains not updated yet | Add the new domain to Auth → Authorized domains BEFORE cutover |

## Verification Checklist

Functional:
- [ ] Anonymous visitor can view every page; cannot see any edit affordances
- [ ] Anonymous visitor cannot write to Firestore (verified by manual `fetch` against Firestore REST API)
- [ ] Non-allowlisted email cannot complete sign-in (signed back out on auth callback)
- [ ] Member can edit text on every page (16 pages)
- [ ] Member can CRUD every list collection (8 collections)
- [ ] Member can upload images + PDFs and see them in the right place on public pages
- [ ] Two members editing same block: second save shows "outdated" prompt, no silent overwrite
- [ ] Admin can add/remove members and change their role
- [ ] Admin can edit site settings (emails, social links); non-admin members cannot

Security:
- [ ] Firestore rules unit tests pass (Phase 3 checklist)
- [ ] Storage rules unit tests pass
- [ ] App Check enforced (verify rejection by hitting Firestore from a script without an App Check token)
- [ ] CSP headers present in response (verify via `curl -I` against the live site)
- [ ] No `javascript:` URLs accepted by the link editor (try in TipTap, should be rejected)
- [ ] Sign-in links only work on authorised domains (try the link on a different host, should fail)
- [ ] No PII (emails, names) appears in browser console or analytics

Performance:
- [ ] Lighthouse Performance ≥90, Accessibility ≥95, SEO ≥95 on home + events + committee
- [ ] Firestore reads for an anonymous visitor on home page ≤ 5 (one per editable block + one per list rendered)
- [ ] No layout shift between HTML default content and Firestore-loaded content

## Out of Scope (for this build)

Carried over from the Payload spec, plus the things we've explicitly deferred:
- Member-only RSVPs / payments
- Email newsletter integration
- Public member directory with self-edit
- Multi-language
- Audit log + draft/publish workflow
- Image swap from the carousel via UI (carousel images stay developer-managed for now — easy additive change later by adding a `homeCarousel/{order}` collection)
- Heading / structure editing (page layout is fixed by the design)
- Bulletin search across past issues

If audit-log or draft/publish become needed, they're additive: add a `pageBlocksHistory/` collection with the same rules, write to it from a Firestore trigger function (would require Blaze plan but still essentially free at this volume).

## Open Questions to Confirm Before Phase 1

1. Email provider for magic links: default Firebase (`noreply@<project>.firebaseapp.com` — works, may go to spam) or SendGrid free tier (better deliverability — free up to 100/day)?
2. Custom domain: keep `kclea.org.uk` apex or move to `www.kclea.org.uk` canonical? (Firebase Hosting supports either; need to know DNS access.)
3. Who is the first admin? (Need their email to seed `members/`.)

---

## Total Estimate

**Calendar time: ~9–19 working days (~2–4 weeks)** assuming a few hours per day of focused work by a non-technical operator using Claude Code.

| Phase | Description | Low (days) | High (days) |
|---|---|---|---|
| 0 | Cloud setup (Firebase project, Auth, Firestore, Storage, App Check, Resize Images extension) | 0.5 | 1.0 |
| 1 | Split SPA into 16 per-page HTML files | 0.5 | 1.5 |
| 2 | Auth scaffolding (passwordless email link + allowlist) | 1.0 | 2.0 |
| 3 | Firestore + Storage security rules + unit tests | 1.0 | 2.5 |
| 4 | In-place text editing with TipTap on all 16 pages | 1.5 | 3.0 |
| 5 | CRUD modal forms for the 8 list collections + site settings | 2.0 | 4.0 |
| 6 | File uploads (PDFs, images) with auto-generated variants | 1.0 | 2.0 |
| 7 | Member management (invite, allowlist UI) | 0.5 | 1.5 |
| 8 | Hardening (CSP, headers), Lighthouse, DNS cutover, SSL | 1.0 | 2.0 |
| | **Total** | **~9 days** | **~19.5 days** |

**Pushes toward the high end:** first time using Firebase, debugging magic-link email deliverability, getting security rules right (Phase 3 is genuinely the riskiest), CORS issues on Storage uploads, DNS cutover surprises.

**Pushes toward the low end:** operator already has a Firebase project from prior work, comfortable with browser dev tools, willing to deploy and test iteratively rather than wait for "complete" before deploying.

**Single biggest schedule risk:** Phase 3 (security rules). If rules are wrong, *every* subsequent phase fails in confusing ways. Budget extra and don't skip the rules unit tests.
