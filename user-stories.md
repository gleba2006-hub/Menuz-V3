# User Stories

_Department: Product & Tech · Agent: Milo Chen · 9/29/2026, 6:57:43 PM_

---

# User Stories: Menuz Tableside (Stage 1)

**Owner:** Milo Chen, Product Manager
**Scope:** Stage 1 pilot (8 weeks, 5 venues). Stage 2 and 3 appear only as backlog epics.
**Sizing key:** XS ≤ 1 day · S ≈ 2–3 days · M ≈ 1 week · L ≈ 2 weeks · XL = split before build

---

## Epic 1: Guest Entry and Menu Browsing

**US-1.1 QR/NFC entry (M)**
As a **guest**, I want to scan a QR code or tap an NFC tag and land on the menu, so that I can start ordering without installing anything.
- Given a QR sticker and an NFC tag on the same table, both open `menuz.app/v/{venue}/t/{tableCode}`.
- The menu shows the venue's name and table number in the header, with no login or account prompt.
- An invalid `tableCode` shows a "Pick your table" fallback instead of an error.

**US-1.2 Fast load on weak networks (M)**
As a **guest in a loud, crowded bar**, I want the menu to load instantly even on poor signal, so that I don't give up and flag down a server.
- Menu is interactive in under 2 seconds on throttled 4G, measured on a mid-range Android device.
- After the first visit, the menu opens from the service-worker cache with no connection, and cart changes queue locally.
- Offline, the "Send order" button shows "Reconnecting…" and never loses the cart.

**US-1.3 Low-light, one-handed design (S)**
As a **guest with a drink in one hand**, I want a dark-mode layout with large tap targets, so that I can order in a dim bar.
- The default theme is dark, with contrast of at least 4.5:1 for body text.
- All primary actions sit in the bottom thumb zone and are at least 48×48 px.

**US-1.4 Allergen and age messaging (S)**
As a **venue owner**, I want allergen tags and an alcohol-age notice shown automatically, so that I stay compliant and reduce risk.
- Items display allergen icons configured in admin, and the guest can filter by "no nuts", "vegetarian" and "gluten-free".
- First add of an alcoholic item shows "Must be 18+ (21+ in the US). ID may be checked." and requires one tap to confirm.
- No personal data is stored when the notice is accepted.

---

## Epic 2: Group Packages ("The Round")

**US-2.1 Choose group size and see live price (M)**
As a **guest ordering for friends**, I want to pick how many people we are and see the package total update, so that I know the cost before committing.
- Stepper accepts 2–12 people, defaulting to 4.
- Price updates in under 100 ms with no network call, using the published snapshot rules.
- Example: 4 people × beer + 1 chips bowl + 1 shareable platter shows an itemized breakdown and a per-person price.

**US-2.2 One-tap add package (S)**
As a **guest**, I want to add the entire package with one tap, so that I can order a full round in under 60 seconds.
- Tapping "Add The Round" adds all items in the correct quantities: beer × N, chips × ⌈N/4⌉, platter × ⌈N/4⌉ (rules are configurable per package).
- The cart shows the package as one grouped line with an "Edit" option.

**US-2.3 Swap items inside a package (M)**
As a **guest**, I want to swap the beer for another allowed option, so that the package fits what my group likes.
- Swaps are limited to the venue's approved list for that slot.
- A swap with a price difference shows the delta (e.g., "+$1.50 per person") before confirmation.
- The package discount is retained after a valid swap.

**US-2.4 Out-of-stock handling (S)**
As a **bartender**, I want to mark an item as 86'd so that guests cannot order it inside a package.
- A staff toggle takes effect on guest menus within 60 seconds.
- Affected packages show a suggested substitute, or are hidden if no valid substitute exists.

---

## Epic 3: Ordering and Payment

**US-3.1 Send order tagged with table (M)**
As a **guest**, I want to submit my order and see a confirmation, so that I know the kitchen has it.
- The server re-validates prices and package rules at submit time, and rejects tampered carts.
- Confirmation shows an order number and an estimated wait time.
- Submitting twice within 5 seconds creates one order (idempotency key).

**US-3.2 Pay now or pay at table (L)**
As a **guest**, I want to pay by card or Apple/Google Pay, or choose to pay the server later, so that I can settle the way I prefer.
- Card entry uses Stripe-hosted elements, so Menuz never stores card data.
- A failed payment keeps the cart and shows a clear retry message.
- "Pay at table" is configurable per venue, and the order is flagged "unpaid" for staff.

**US-3.3 Add a round later (S)**
As a **guest**, I want to reorder the same package in one tap, so that the next round is quick.
- A "Same again" button appears after the first order in the same table session (Redis-backed, 4-hour expiry).
- The new order is a separate ticket linked to the same table.

---

## Epic 4: Staff Order Handling

**US-4.1 Live order feed (M)**
As **Priya, a server**, I want new orders to appear on my phone with table numbers, so that I don't do double entry.
- Orders appear via SSE within 3 seconds and trigger a sound and visual alert.
- Each ticket shows table, items, package grouping, notes and paid/unpaid status.
- Staff can mark orders Accepted → Ready → Served.

**US-4.2 Print and email fallback (M)**
As **Marco, a GM**, I want orders to print on the kitchen printer or arrive by email if the staff view is unavailable, so that no order is lost.
- Star CloudPRNT printers receive the ticket within 10 seconds of submission.
- Failed deliveries retry 3 times, then land in a dead-letter queue and alert Menuz support.

---

## Epic 5: Venue Configuration (Menuz Staff)

**US-5.1 Configure a venue from a template (L)**
As a **Menuz onboarding specialist**, I want to build a venue's menu from a template, so that I can go live within 2 hours of configuration.
- I can set logo, colors, fonts, categories, items, photos, allergens and prices.
- The admin tool requires Cognito login with MFA.
- A pilot venue is configured in under 2 hours by someone who did not build the tool.

**US-5.2 Build and publish packages (M)**
As a **Menuz onboarding specialist**, I want to define package rules (slots, per-person items, shared items, sizing) so that "The Round" can be tailored per venue.
- Rules support "per person", "per 3–4 people" and fixed-price options.
- A preview simulates group sizes 2–12 and flags any price anomalies.

**US-5.3 Draft, publish and roll back (M)**
As **Dana, a bar owner**, I want changes to go live only when approved, with an undo option, so that a mistake never reaches guests.
- Publish creates a versioned JSON snapshot on CloudFront, live within 2 minutes.
- Rolling back to any of the last 10 versions takes one click.

**US-5.4 Generate QR and NFC assets (S)**
As a **Menuz specialist**, I want to export table QR codes and NFC URLs in bulk, so that I can prepare venue kits quickly.
- I can export a PDF of QR stickers labeled by table, plus a CSV of NFC URLs.

---

## Epic 6: Owner Insights

**US-6.1 Weekly uplift report (M)**
As **Dana, a bar owner**, I want a weekly email showing package attach rate and average check versus baseline, so that I can see Menuz paying for itself.
- The report shows scans, order-start rate, package attach rate, average check and seating-to-first-order time.
- Analytics collect no personal data or persistent guest identifiers.

---

## Backlog Epics (Not Estimated)

| Epic | Stage | Notes |
|---|---|---|
| POS order transmission (two adapters, e.g., Square, Toast) | 2 | Gate: ≥ 3 owner requests and ≥ 99% accuracy |
| Owner self-serve menu editor | 2 | Follows pilot feedback |
| Cinematic table projection synced to menu | 3 | Research only, requires paid pilot commitment |

---

## Sizing Summary

| Epic | Stories | Approx. effort |
|---|---|---|
| 1. Entry and browsing | 4 | 1 M + 1 M + 2 S ≈ 2.5 weeks |
| 2. Packages | 4 | 2 M + 2 S ≈ 3 weeks |
| 3. Ordering and payment | 3 | 1 L + 1 M + 1 S ≈ 3.5 weeks |
| 4. Staff handling | 2 | 2 M ≈ 2 weeks |
| 5. Configuration | 4 | 1 L + 2 M + 1 S ≈ 4 weeks |
| 6. Insights | 1 | 1 M ≈ 1 week |

**Total: ≈ 16 person-weeks against a 24 person-week squad capacity.** The buffer covers pilot fixes and QA on real devices in real bars.