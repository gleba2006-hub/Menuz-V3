# Executive Delegation Brief

_Department: Executive · Agent: Ada Voss · 9/29/2026, 6:36:10 PM_

---

# Executive Delegation Brief: Menuz Tableside

**From:** Ada Voss, CEO
**To:** Product & Tech team (Head of Product, CTO, and specialist owners)
**Workflow:** Product Development SOP
**Status:** Approved for Discovery → Build (Stage 1)

---

## 1. Interpreting the Mission

The mission reads "help us buy ld menus." I am reading it as **custom-made digital menus that owners can launch quickly**, opened by QR scan or NFC tap. Anyone who reads it differently should flag it to me within 24 hours. Otherwise this is the working definition.

We sell to **restaurant and bar owners**, and our differentiator is **menus that sell**, not menus that only display. Each menu is built around pre-configured group packages that raise the average check and shorten the time to order.

---

## 2. Objective

Deliver a three-stage product roadmap. Stage 1 must be live in **pilot venues within 8 weeks**.

| Stage | What we ship | Outcome for the owner |
|---|---|---|
| **1. Selling Menu** | Custom mobile web menu opened by QR/NFC, with **per-headcount packages** and one-tap ordering | Higher average check, faster ordering |
| **2. Table-to-Kitchen** | Integration with restaurant POS/ordering software, so each table's order goes straight to the kitchen/bar | Fewer staff steps, fewer errors |
| **3. Cinematic Table** | Table-projected cinematic video, synced to the menu | A signature experience that justifies premium venues and pricing |

**Stage 1 hero example ("The Round"):** for a group of N, the package includes 1 beer per person + a shared chips bowl + 1 shareable platter sized for 3–4 people. The guest picks group size and sees a live price. One tap adds the package to the order.

---

## 3. Success Metrics

**Stage 1 (measured at day 60 of pilot):**

| Metric | Target |
|---|---|
| Pilot venues live | 5 (mix of bars and casual restaurants) |
| Menu scan → order start rate | ≥ 35% |
| Package attach rate (orders containing a package) | ≥ 30% |
| Average check uplift vs. venue baseline | ≥ 12% |
| Time from table seating to first order | ≥ 25% faster |
| Menu load time on 4G | < 2 seconds |
| Owner setup time (menu live from our intake) | ≤ 48 hours |
| Pilot owner NPS | ≥ 40 |

**Stage 2 gate:** ≥ 3 pilot owners request POS integration and order-transmission accuracy is ≥ 99%.
**Stage 3 gate:** a paid pilot commitment from at least 1 venue before we invest in hardware.

---

## 4. Constraints

- **No app download.** Everything runs in the mobile browser, and NFC and QR must open the same URL.
- **Budget:** Stage 1 build cost is capped at what a 3-person squad can deliver in 8 weeks. No custom hardware until Stage 3.
- **Bars are dark, loud, and busy.** Design for one-handed use, low light, and spotty Wi-Fi, with graceful degradation on weak connections.
- **Per-venue customization is a template system, not bespoke code.** Every "custom" menu must be configurable in under 2 hours by our team.
- **POS compatibility (Stage 2):** start with the two most common systems among pilot venues (candidates: Square, Toast, Lightspeed) rather than trying to support everything.
- **Compliance:** allergen and alcohol-age messaging, GDPR/CCPA-safe analytics with no personal data required to order, and PCI-compliant payment handling. We use a third-party processor and never store card data.
- **Stage 3 risk:** ambient light, table surface, and projector mounting vary widely. Stage 3 stays in research only until Stage 1 proves demand.

---

## 5. Ownership

| Owner | Responsibility | Key deliverables |
|---|---|---|
| **Head of Product** | Overall Stage 1 scope, pilot selection, prioritization | PRD, package logic rules, pilot venue list, go/no-go for each stage gate |
| **UX/UI Designer** | Mobile ordering flow, group-size selector, visual system | Wireframes in week 2, clickable prototype in week 3, venue theme kit |
| **Menu Merchandising Specialist** | Package design and pricing logic (beer + chips + shareable per group size) | Package templates for bars, pubs, and casual dining, plus upsell copy |
| **Frontend Engineer** | Mobile web menu, QR/NFC entry, cart and package builder | Production-ready menu web app by week 6 |
| **Backend Engineer** | Menu/venue data model, order API, admin panel | Multi-venue backend, order log, menu config tool |
| **Integrations Engineer** | POS research and Stage 2 architecture | POS integration spike report by week 6, with recommendation and effort estimate |
| **Data Analyst** | Event tracking and dashboards | Scan → view → package → order funnel, per-venue weekly report |
| **QA Engineer** | Device/browser matrix, NFC tag testing, load testing | Test plan, NFC compatibility list (iOS/Android), release sign-off |
| **Creative Technologist** | Stage 3 feasibility only | Projector/video research memo and cost model, no build |

**CTO** owns architecture decisions, security and payment compliance, and technical risk. **Head of Product** is the single point of accountability to me.

---

## 6. Timeline and Checkpoints

- **Week 1:** Discovery kickoff, pilot venue shortlist, package logic draft.
- **Week 2:** Wireframes and data model approved.
- **Week 4:** Internal demo of working menu with one real venue's data.
- **Week 6:** Feature freeze, QA begins, POS spike report delivered.
- **Week 8:** Pilot launch in 5 venues.
- **Day 60:** Metrics review with me, then Stage 2 go/no-go.

I want a **written 1-page status every Friday**: progress, risks, decisions needed.

---

## 7. Open Decisions Needed From Product (by end of Week 1)

1. Do we take payment in the menu at Stage 1, or send orders to staff for table payment? My default is **order without payment** to ship faster.
2. Which drinks and food categories are in the first package templates? My default is beer, chips, and shareable platters, then cocktails in a later release.
3. Who supplies photography and menu content for pilot venues: the owner or us?

Bring me recommendations, not questions. Ship the simplest version that proves the package model sells.

**— Ada Voss, CEO, Menuz**