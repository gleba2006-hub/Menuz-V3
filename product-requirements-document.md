# Product Requirements Document

_Department: Product & Tech · Agent: Milo Chen · 9/29/2026, 6:36:35 PM_

---

# Product Requirements Document: Menuz Tableside

**Owner:** Milo Chen, Product Manager
**Status:** Draft v1.0 for Engineering, Design, and CEO review
**Scope:** Stage 1 (detailed), Stages 2–3 (directional)
**Interpretation note:** Per Ada Voss's brief, "buy ld menus" is read as *custom-made digital menus launched quickly by restaurant and bar owners*.

---

## 1. Problem

Owners of bars and casual restaurants lose revenue at the table in three ways:

- **Slow first order.** Guests wait 8–15 minutes for a server on busy nights, and some leave or order less.
- **Menus that list instead of sell.** PDFs and laminated cards make groups assemble their own order item by item, so they under-order, especially shareables.
- **Weak digital options.** Existing QR menus are static PDFs, and full ordering platforms take weeks to configure and cost more than small venues will pay.

**Opportunity:** a template-driven mobile menu that bundles items into per-headcount packages, so a group orders a full round in one tap.

---

## 2. Goals and Non-Goals

### Goals (Stage 1, 8 weeks to pilot)
1. Launch **5 pilot venues** with a live, branded mobile menu opened by QR or NFC.
2. Raise **average check ≥ 12%** through package selling.
3. Cut **seating-to-first-order time ≥ 25%**.
4. Let Menuz staff configure any venue in **under 2 hours**, with the menu live **within 48 hours** of intake.

### Non-Goals (Stage 1)
- No native app and no guest account or login.
- No POS integration (Stage 2). Orders go to a simple staff view and printer or email fallback.
- No projector or video hardware (Stage 3, research only).
- No owner self-serve builder. Menuz staff configure venues through an internal admin tool.
- No loyalty, delivery, or reservations.

---

## 3. Target Users and Personas

| Persona | Profile | Needs | Success looks like |
|---|---|---|---|
| **Dana, Bar Owner** (buyer) | Runs a 60-seat craft beer bar, 2 locations, thin margins | Higher check, fewer idle guests, no tech burden | Sees weekly uplift report, menu changes done in minutes |
| **Marco, Restaurant GM** (buyer and admin) | Casual dining, 120 seats, manages 15 staff | Fewer order errors, faster table turns | Staff spend less time on simple drink rounds |
| **Priya, Server / Bartender** (operator) | Handles 6–8 tables on a Friday night | Clear incoming orders, no double entry | Orders arrive tagged with table number |
| **Groups of 3–6 friends** (end guest) | Phones in hand, dim lighting, one person orders for all | Fast decisions, clear total, no sign-up | Orders a round in under 60 seconds |

---

## 4. Competitive Landscape

| Competitor type | Examples | Strengths | Gaps we exploit |
|---|---|---|---|
| POS-native ordering | Toast Order & Pay, Square Online | Deep POS integration | Locked to their POS, generic menus, no group-package selling |
| QR menu tools | Menu Tiger, Sirved-style PDF QR | Cheap and fast | Static display, no upsell logic, no analytics tied to revenue |
| Table-order platforms | Mr Yum, Ordermark | Full ordering | Heavy setup, limited customization, pricing aimed at chains |
| Experiential tech | Projection-dining ventures (Le Petit Chef) | Wow factor | Bespoke, expensive, not linked to ordering |

**Positioning:** Menuz is the only *menu built to sell to groups*, launched in 48 hours, with a path to POS sync and cinematic tables.

---

## 5. Core Features (Prioritized)

### P0: Must ship for pilot
| ID | Feature | Description |
|---|---|---|
| F1 | QR/NFC entry | Both open the same venue URL with table ID parameter; no app |
| F2 | Template menu renderer | Venue theme (logo, colors, fonts), categories, items, photos, dark-mode default |
| F3 | Group Packages ("The Round") | Guest selects headcount; package scales (1 beer/person + chips bowl + 1 shareable per 3–4 people); live price |
| F4 | One-tap order | Add package or items, review, submit, with table number attached |
| F5 | Staff order view | Live web dashboard with sound alert, status (new/preparing/served), email/print fallback |
| F6 | Payments handoff | Third-party processor (e.g., Stripe) or "pay at table" option; no card data stored |
| F7 | Compliance messaging | Allergen tags, 18+/21+ age acknowledgment for alcohol |
| F8 | Internal admin tool | Menuz staff create venues, packages, and rules from templates |

### P1: Should ship in pilot if capacity allows
- Package swap rules (e.g., swap beer for a soft drink or cider)
- Sold-out toggle by staff
- Basic analytics dashboard (scans, package attach rate, check size)
- Multi-language toggle (EN plus one local language)
- Offline-tolerant loading (cached menu, queued submit)

### P2: Later
- Owner self-serve editor
- Time-based packages (happy hour)
- Reorder-a-round button
- Stage 2 POS connectors (Square, Toast, or Lightspeed, chosen from pilot data)
- Stage 3 projector research

---

## 6. Requirements Pool

| ID | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| R1 | Menu loads on 4G | P0 | Largest Contentful Paint < 2s at p75 on mid-range Android |
| R2 | One-handed, low-light use | P0 | Tap targets ≥ 48px, contrast ≥ WCAG AA, dark theme default |
| R3 | Package price recalculates instantly | P0 | Changing headcount updates total in < 200ms, no reload |
| R4 | Package scaling rule | P0 | Shareables = ceil(headcount ÷ 4), configurable per venue |
| R5 | No login to order | P0 | Order completes with zero personal data; optional name for pickup |
| R6 | Order delivery reliability | P0 | ≥ 99% of submitted orders appear in staff view within 5s |
| R7 | Weak connection handling | P1 | Submit retries automatically; guest sees clear status |
| R8 | Analytics privacy | P0 | No cookies requiring consent, no personal identifiers |
| R9 | Venue setup speed | P0 | Staff configure a venue in ≤ 2 hours from template |
| R10 | Age gate | P0 | Alcohol items require confirmation before checkout |
| R11 | Menu editing | P1 | Price or availability change live in < 1 minute |
| R12 | Stage 2 readiness | P2 | Order payload is structured JSON with item IDs mappable to POS SKUs |

---

## 7. Success Metrics (Day 60 of pilot)

| Metric | Target |
|---|---|
| Pilot venues live | 5 |
| Scan → order start | ≥ 35% |
| Package attach rate | ≥ 30% |
| Average check uplift | ≥ 12% |
| Seating → first order | ≥ 25% faster |
| Load time on 4G | < 2s |
| Setup time from intake | ≤ 48h |
| Owner NPS | ≥ 40 |

**Stage gates:** Stage 2 needs ≥ 3 owners requesting POS integration and ≥ 99% order accuracy. Stage 3 needs 1 paid venue commitment.

---

## 8. Open Questions

| # | Question | Owner | Needed by |
|---|---|---|---|
| 1 | Which payment model for pilot: pay at table, in-app, or both? | Milo and CTO | Week 1 |
| 2 | Which POS systems do the 5 pilot venues use? This decides the Stage 2 targets. | Sales | Week 2 |
| 3 | What are the legal alcohol-ordering rules by pilot region (e.g., can guests order alcohol via phone)? | Legal | Week 1 |
| 4 | How is the average check baseline measured without POS access? | Data | Week 2 |
| 5 | Are package prices fixed, or may owners discount vs. à la carte? | Milo and pilot owners | Week 2 |
| 6 | How do we prevent fake or prank orders from outside the venue (table token, geofence)? | CTO | Week 3 |
| 7 | Pricing model for owners: flat monthly or revenue share? | CEO | Week 4 |
| 8 | NFC tag hardware sourcing and cost per table? | Ops | Week 2 |