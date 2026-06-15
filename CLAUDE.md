# LAAT Custom Website — AI Instructions

This is a standalone HTML website for the **Liberian Association of Austin Texas (LAAT)**.
It is a static site (`laat-austin_1.html`) that pulls live data from the organization's
AlkeLedger workspace and links members to the right pages in the app.

---

## AlkeLedger Setup

| Item | Value |
|------|-------|
| Org slug | `laat` |
| App base URL | `https://app.alkeledger.com` |
| Public data API | `https://app.alkeledger.com/api/public-data?orgSlug=laat` |

The `ORG_SLUG` and `APP_URL` variables at the top of the integration script (inside the
`<script>` block) control all of this. **Change only those two variables if the org ever
moves to a new slug or domain.**

---

## Live Data Integration

A JavaScript block at the bottom of the `<script>` tag fetches
`https://app.alkeledger.com/api/public-data?orgSlug=laat` on page load.

The API returns:

```json
{
  "memberCount": 42,
  "upcomingEvents": [
    {
      "id": "abc123",
      "title": "Monthly General Meeting",
      "startDate": "2026-07-12T18:00:00",
      "endDate": null,
      "location": "Austin Central Library",
      "description": "Quarterly updates...",
      "allDay": false
    }
  ],
  "duesRates": {
    "individual": 60,
    "organization": 250
  },
  "transparencyUrl": "https://app.alkeledger.com/laat/transparency",
  "eventsUrl": "https://app.alkeledger.com/laat/events"
}
```

### What gets updated automatically

| HTML element | ID / class | Data used |
|---|---|---|
| Stats bar member count | `id="stats-members"` | `memberCount` |
| Events grid | `id="laat-events-grid"` | `upcomingEvents` (replaces hardcoded cards) |
| Standard membership price | `id="laat-price-individual"` | `duesRates.individual` |
| Business Partner price | `id="laat-price-corporate"` | `duesRates.organization` |
| "Access Portal" button(s) | `class="laat-portal-link"` | `transparencyUrl` |

The hardcoded content in these elements acts as the fallback if the API fails or is slow.
Do not remove the fallback values.

### Rendering events

Each event from `upcomingEvents` is rendered as an `.event-card` div. The RSVP link for
each card points to:
```
https://app.alkeledger.com/share/laat/event/{event.id}
```
This is the public share/RSVP page for that specific event — no login required to view it.
Members who click RSVP will be prompted to sign in if not already authenticated.

---

## AlkeLedger Page URLs

Use these URLs for action buttons and links. Always open in a new tab (`target="_blank"`).

| Action | URL |
|--------|-----|
| Sign in / access workspace | `https://app.alkeledger.com/laat` |
| View events calendar | `https://app.alkeledger.com/laat/events` |
| Financial transparency | `https://app.alkeledger.com/laat/transparency` |
| Member directory | `https://app.alkeledger.com/laat/members` |
| Governance documents | `https://app.alkeledger.com/laat/documents` |
| Pay dues / membership | `https://app.alkeledger.com/laat/dues` |
| Announcements | `https://app.alkeledger.com/laat/announcements` |
| RSVP to a specific event | `https://app.alkeledger.com/share/laat/event/{eventId}` |

---

## Button Wiring Guide

### Requires AlkeLedger (link to app, open `target="_blank"`)

| Button / link | Target URL |
|---|---|
| "Join LAAT" / "Join Now" (membership) | `…/laat` |
| "Apply" (corporate membership) | `…/laat` |
| "View Full Calendar" | `…/laat/events` |
| Event "RSVP" buttons | `…/share/laat/event/{id}` (set dynamically by JS) |
| "Access Portal" (transparency) | `…/laat/transparency` (set dynamically by JS) |
| "Annual Financial Reports" card | `…/laat/transparency` |
| "Meeting Minutes" card | `…/laat/transparency` |
| "Budget Reports" card | `…/laat/transparency` |
| "Governance Documents" card | `…/laat/documents` |
| "Our Constitution" | `…/laat/documents` |
| "Meet Leadership" | `…/laat/members` |
| Footer — Leadership Team | `…/laat/members` |
| Footer — Constitution & Bylaws | `…/laat/documents` |
| Footer — Meeting Minutes | `…/laat/transparency` |
| Footer — Financial Reports | `…/laat/transparency` |
| Footer — Events Calendar | `…/laat/events` |

### Stays on this page (anchor links or contact form)

| Button / link | Target |
|---|---|
| "Become a Volunteer" | `#contact` |
| "Apply for Scholarship" | `#contact` |
| "Request Support" (welfare) | `#contact` |
| "Get Listed" (business directory) | `#contact` |
| Donate buttons | `#contact` (or payment link when set up) |
| Social media links | External social URL when available |

---

## How the Backend API Works

The `publicOrgData` function lives in `functions/src/index.ts` in the AlkeLedger repo.
It is a Firebase Cloud Function (v2, `onRequest`, CORS enabled) that:

1. Accepts `?orgSlug=laat`
2. Looks up the org by slug in Firestore
3. Queries active memberships count
4. Queries upcoming events (next 6, ordered by `startDate` ascending)
5. Reads `duesRates` from the org document
6. Returns JSON with all of the above + convenience URLs

It is routed via Firebase Hosting rewrite: `GET /api/public-data` → `publicOrgData`.
Response is cached for 5 minutes (`Cache-Control: public, max-age=300`).

The function uses the Firebase Admin SDK so it bypasses Firestore security rules —
**no authentication is required from the website visitor**.

---

## What's Not in AlkeLedger — Needs Client Input

These parts of the site are currently placeholders. An AI cannot fill them in from
AlkeLedger data — they require information directly from LAAT.

### Social Media Links
All social icons in the contact section and footer are `href="#"`.
Replace with the actual URLs once LAAT provides them:

| Platform | Where in HTML | Replace `href="#"` with |
|----------|--------------|------------------------|
| Facebook | Contact section + footer | LAAT Facebook page URL |
| Instagram | Contact section | LAAT Instagram URL |
| Twitter/X | Contact section | LAAT Twitter/X URL |
| YouTube | Contact section | LAAT YouTube channel URL |

### Donation / Payment Buttons
The three donate buttons ("One-Time Donation", "Monthly Support", "Sponsorship Package")
have no payment destination. Options when client is ready:
- **Venmo / CashApp / Zelle**: link to their handle or phone
- **PayPal.Me**: `https://paypal.me/LAATAUSTIN`
- **Stripe / Square**: link to their hosted payment page
Until then, redirect them to `#contact` so visitors can reach out.

### Contact Form
The contact form (`#contact`) does not submit anywhere. Wire it up using one of:
- **Formspree**: add `action="https://formspree.io/f/{formId}"` to the `<form>` tag
- **EmailJS**: add their SDK and call `emailjs.send(...)` on submit
- **Direct mailto fallback**: change the submit button to `<a href="mailto:info@laataustin.org">`
Get the correct email address from LAAT before wiring this up.

### President's Name and Photo
The president section uses a placeholder avatar (👤 emoji) and a generic title.
Replace when LAAT provides:
- President's full name → update `<p class="president-name">`
- President's title/term → update `<p class="president-title">`
- Photo: replace the emoji `div.president-photo` with an `<img>` tag

### Family Membership Price ($100)
AlkeLedger's `duesRates` only stores `individual` and `organization` rates.
The Family tier price (`$100 / year`) is **hardcoded** and will not update automatically.
If LAAT changes their family rate, it must be updated manually in the HTML:
```html
<div class="mem-price" style="color:#FFC94D;">$100 <span ...>/ year</span></div>
```

### "Join LAAT" / Membership Buttons
Currently these link to `https://app.alkeledger.com/laat`, which requires an invite code
to join. If LAAT wants new members to join directly from the website, they should either:
- Share a public invite link (`app.alkeledger.com/join/{inviteCode}`) — get the code
  from their AlkeLedger Settings page and use it as the `href` on these buttons
- Or redirect to `#contact` so interested members can email LAAT for an invite

### Logo / Branding
The nav badge is text-only ("LAAT"). If LAAT has a logo image, replace:
```html
<div class="nav-logo-badge">LAAT</div>
```
with an `<img>` tag pointing to their logo file.

### Footer Copyright Year
```html
© 2025 Liberian Association of Austin Texas
```
Update the year as needed. Consider making it dynamic:
```html
© <span id="footer-year"></span> Liberian Association of Austin Texas
```
```js
document.getElementById('footer-year').textContent = new Date().getFullYear();
```

---

## Development Notes

- The file is a **self-contained single HTML file**. Keep it that way — no build step,
  no npm, no bundler.
- All CSS is inline in `<style>`. All JS is inline in `<script>`.
- The AlkeLedger integration script is wrapped in an IIFE at the bottom of `<script>`.
  The `ORG_SLUG` and `APP_URL` at the top of that IIFE are the only config values.
- If data fails to load, the hardcoded placeholder content remains visible. This is
  intentional progressive enhancement — the site works without the API.
- Font: Playfair Display (display) + Outfit (body) via Google Fonts.
- Colors: `--red: #BF0A30`, `--blue: #003897`, `--gold: #C49A00` (Liberian flag palette).

---

## Deploying AlkeLedger Changes

If you modify `functions/src/index.ts` or `firebase.json` in the AlkeLedger repo:

```bash
# Deploy only the public data function
firebase deploy --only functions:publicOrgData

# Deploy function + hosting rewrite together
firebase deploy --only functions:publicOrgData,hosting
```

Changes to `laat-austin_1.html` are deployed separately by the client to their own
web host — they are **not** part of the AlkeLedger Firebase Hosting deployment.
