# KCLEA Website CMS — Conclusion

**For:** KCLEA Committee · **Date:** 5 August 2026
**Full detail:** [technical plan](./2026-05-13-kclea-firebase-cms-design.md) · [stakeholder summary](./2026-07-11-kclea-cms-stakeholder-summary.md)

---

## The conclusion

**The plan works, it costs nothing to run, and we recommend proceeding.**

We can add a "log in and edit" layer to the new KCLEA website — the same experience the committee already knows from Squarezone. Committee members sign in with a one-time link sent to their email (no passwords), then edit text, events, the Bulletin, the committee roster and the archives directly on the page. To the public, the site looks and loads exactly as it does today.

**Cost: £0 per month, permanently.** It runs on Google Firebase's free tier — no credit card, no server, no monthly bill. The free allowances are roughly 100× what KCLEA will use.

**Time: 2–4 weeks** of build work, delivered in phases with a visible checkpoint every few days. Nothing is hidden until launch.

**Security:** only emails on an approved list can sign in; there is no public sign-up. The one genuinely risky step is the permission rules that decide who may change what — those are tested before anything ships.

---

## The one trade-off to be aware of

Because edits load in the visitor's browser (this is what keeps it free), **Google search results and LinkedIn/WhatsApp link previews may show the original page text rather than the latest committee edits.**

For an association that reaches its members by email rather than through Google search, we believe this is an acceptable trade. Removing the limitation would require a paid plan (~£5–20/month). **This is the committee's call and we need an explicit yes or no.**

---

## What we need to start

Four short answers:

1. **First administrator** — whose email seeds the account? *(suggested: the President)*
2. **Magic-link sender** — Firebase default *(recommended, zero setup)* or SendGrid?
3. **Web address** — `kclea.org.uk` *(recommended)* or `www.kclea.org.uk` as the main one?
4. **The trade-off above** — free tier is fine, or pay for full search/share accuracy?

Once those are confirmed, we begin.

---

## What it deliberately will not do

Page layouts and new sections stay with the developer — committee members edit text, lists and photos only. There is no draft mode (Save publishes immediately) and no undo history, so deletions are permanent. Payments, RSVPs and a public member directory are separate future projects.
