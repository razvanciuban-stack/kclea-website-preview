# KCLEA Website CMS — Stakeholder Summary

**Prepared for:** KCLEA Committee
**Date:** 11 July 2026
**Reference:** [Firebase CMS implementation plan](./2026-05-13-kclea-firebase-cms-design.md) (technical)

---

## Bottom line

We can give the committee a **"log in and edit"** panel on top of the new website — same "Squarezone-familiar" experience the committee already knows. **No monthly bill, no credit card, no server to look after.** Build takes roughly **2–4 weeks**, and there are **four small decisions** the committee needs to make before we start.

---

## What the committee will get

A small **members-only layer** sitting on top of the new site. To the public, nothing changes — the site looks identical to what you can already preview at <https://razvanciuban-stack.github.io/kclea-website-preview/>.

Once signed in, a committee member can:

| Action | How it works |
|---|---|
| **Edit text on any page** | Click a paragraph, it turns into a simple editor (bold, italic, links, lists), change it, click **Save** |
| **Manage upcoming events** | Add / edit / delete events via a pop-up form |
| **Publish a new Bulletin issue** | Upload the PDF + cover image + issue details |
| **Update the Committee roster** | Add / remove / re-order members, change their role, upload a photo |
| **Maintain the archives** | Past Annual Lectures, 13 Club Trophy winners, Bursary winners, Useful Links |
| **Record In Memoriam entries** | Add a new name, dates |
| **Edit contact details / social links** | The President, Bulletin, and Alumni Office emails; LinkedIn URLs; the Investment Fund address |

Everything happens **in place on the actual page** — no separate admin dashboard to learn.

---

## How signing in works

1. On the login page, a committee member types their email
2. They receive a one-time **"magic link"** by email (no password)
3. They click it, it opens the site, and they're signed in
4. Small **"Edit"** buttons and toolbars appear where they can make changes
5. When they close the browser or click "Sign out", the edit affordances disappear

Only emails on an **approved allowlist** can sign in — even someone who intercepts a magic link cannot get in unless the admin has already added them. Random visitors cannot create accounts.

---

## Cost

**£0 per month, forever.** Uses Google Firebase's free tier ("Spark plan"). The free limits are **at least 100 times bigger** than KCLEA will realistically use:

- 50,000 page loads with edits per day (KCLEA needs a handful)
- 20,000 saves per day
- 5 GB of hosting bandwidth per month
- 1 GB of file storage (roughly 100 committee photos + 20 Bulletin PDFs)

**No credit card is required.** No monthly bill will ever appear. The only cost is the developer time to build it (already discussed separately).

---

## What we need from the committee (4 decisions)

### 1. Who is the first admin? *(email)*

We need one committee member's email to seed as the initial "administrator". They'll be able to add / remove other members afterwards.

Suggested: **the President** (Peter Weitzel) or a designated technical lead.

### 2. Magic-link email sender

Two options — both free:

- **Firebase default** (`noreply@kclea-prod.firebaseapp.com`) — zero setup. Magic links may go to spam the first time (each recipient marks "Not Spam" once, then future links arrive normally).
- **SendGrid free tier** — takes ~half a day to set up, delivers cleanly from `noreply@kclea.org.uk`.

Given the committee is 5–10 people who sign in rarely, we recommend **Firebase default** — spam only bites once per person.

### 3. Domain setup

Should the new site live at `kclea.org.uk` (apex) with `www.kclea.org.uk` redirecting to it, or the other way round? Either works; we just need to know before DNS cutover.

**Recommendation:** apex (`kclea.org.uk`) with `www` → apex redirect. Cleaner URLs, same behaviour.

### 4. ⚠️ SEO & social-share previews — the honest one

This is the one non-obvious limitation and it needs the committee's explicit "yes, that's acceptable" **before we build**.

**How it works technically:** the CMS loads edited content into visitors' browsers *after* the page arrives. This is what keeps the whole thing free.

**What that means in practice:**
- Google may not always index the very latest committee-edited text
- Sharing an event link on LinkedIn or WhatsApp may show a **generic page preview** rather than the specific event's title / image
- Very recent changes may take a few days to appear in search results

**Two answers are acceptable — pick one:**

| Answer | Implication |
|---|---|
| **A. That's fine** — search / share previews aren't critical for KCLEA | Proceed as planned. **£0/month.** |
| **B. We need every edit indexable and every event to share cleanly** | Requires paid Firebase plan (Blaze) with pre-rendering. Roughly **£5–20/month depending on traffic.** No committee funding needed today, but this is a monthly bill. |

**Our reading:** for an alumni association that shares events by email to its own members (rather than trying to reach random Google searchers), **A. is almost certainly fine**. But this is the committee's call, not ours.

---

## Timeline

**2 to 4 weeks** of focused build work, phased so we can review together as we go.

| Week | What happens |
|---|---|
| **Week 1** | Firebase account created · Site split into per-page files · Login system built |
| **Week 2** | Security rules · Text editing rolled out across all pages |
| **Week 3** | Add/edit forms for events, bulletin, committee, in memoriam, etc. · Photo & PDF upload |
| **Week 4** | Final security checks · Custom domain (`kclea.org.uk`) cutover · Committee training |

Each phase ends with a working checkpoint — nothing is "hidden until launch". You'll see progress every few days.

---

## What this deliberately does NOT do

Setting expectations up front:

- **Committee members cannot change the page *layout* or add new sections** — only edit the text / lists / photos that already exist. This is intentional — it keeps the design consistent and prevents accidental damage.
- **No live simultaneous editing.** If two people edit the same paragraph at once, the second person to save gets a *"this was just changed — reload?"* prompt. No silent overwrites.
- **No "draft" mode.** Clicking **Save** publishes immediately. If you want to compose privately first, do it in a text file and paste in.
- **No change history / "undo" beyond the current session.** An accidental delete isn't recoverable. We'll add a written "delete confirms with a prompt" safeguard.
- **No payments, RSVPs, or public member directory** — those are separate future projects.
- **Adding a new committee member takes a small 2-step manual process.** They log in once (which fails because they're not on the allowlist yet), the admin adds their ID to the allowlist, and they log in again for real. Takes about 5 minutes and happens rarely.

---

## What could go wrong (honest heads-up)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **Security rules bug** — the code that decides "can this user write this record?" is easy to get subtly wrong | Medium | Site becomes either insecure or quietly broken | Unit tests must pass before we ship. This is the single riskiest step and we treat it accordingly. |
| Magic-link emails land in spam initially | High (first time only) | Recipient has to fish it out once | Firebase default works; SendGrid available if needed |
| Google Firebase free tier limits are exceeded | Very low | Reads temporarily paused (page keeps serving its baked-in HTML) | Would only happen with sudden viral traffic; we'd upgrade to paid tier reactively |
| The person doing the build gets stuck | Low | Delay of a day or two | The plan has explicit "you should see X" checkpoints per phase — mistakes are caught early, not at the end |

---

## Ownership & handover

Everything the committee needs to own is under KCLEA-controlled accounts:

- **The domain** — `kclea.org.uk` at the current registrar (KCLEA already owns this)
- **The code** — GitHub repository (currently on the developer's account; can transfer to a KCLEA organisation at any time — 2-minute operation)
- **The Firebase project** — created under a KCLEA-designated Google account
- **All member data + PDFs + photos** — inside the Firebase project

**No vendor lock-in.** The site is standard HTML / CSS / JavaScript; if the committee ever wants to leave Firebase, everything is portable.

---

## What we need from the committee to start

Please confirm the following four items and we begin Phase 0:

- [ ] **First admin email:** ______________________________
- [ ] **Magic-link sender:** ☐ Firebase default (recommended) ☐ SendGrid
- [ ] **Domain form:** ☐ `kclea.org.uk` apex (recommended) ☐ `www.kclea.org.uk` canonical
- [ ] **SEO/share previews decision:** ☐ A — free tier is fine ☐ B — paid tier for pre-rendering
- [ ] **Approved to begin build:** ☐ Yes

---

*The full technical plan (with day-by-day phases, code excerpts, and security rule text) is in the same repository at `docs/superpowers/specs/2026-05-13-kclea-firebase-cms-design.md`. This summary is written for committee members; the technical plan is written for the developer.*
