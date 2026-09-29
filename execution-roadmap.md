# Execution Roadmap

_Department: Product & Tech · Agent: Rafa Lindqvist · 9/29/2026, 6:58:13 PM_

---

# Execution Roadmap: Menuz Tableside

**Owner:** Rafa Lindqvist, Engineering Lead
**Inputs:** Executive Delegation Brief, PRD v1.0, System Architecture, Stage 1 User Stories
**Horizon:** Stage 1 in detail (Weeks 1–8), Stages 2–3 directional

---

## 1. Planning Assumptions

- Team is a 3-person squad on 4 two-week sprints, with a demo every second Friday.
- Story sizes use Milo's key (XS ≤ 1 day up to L ≈ 2 weeks). Capacity is about 13 engineer-weeks after 20% overhead for review, deploys and pilot support.
- Scope is protected by cutting P1 items first. **The 8-week date and the 5 pilot venues do not move.**
- Pilot venues are signed by Week 3 (Milo and Ada own this).

---

## 2. Phased Milestones

| Milestone | Weeks | Exit criteria |
|---|---|---|
| **M0: Walking skeleton** | 1–2 | Staging and prod in Terraform. CI/CD runs. A hardcoded menu snapshot on CloudFront opens from a QR URL on a real phone. |
| **M1: Sellable menu** | 3–4 | Guest PWA renders the snapshot, "The Round" calculator and one-tap add work with live pricing, and the cart is offline-safe. Internal demo. |
| **M2: Orderable** | 5–6 | Stripe payment, order → SQS → staff live view, email and Star printer fallback. Admin tool can create a venue and publish a snapshot. **First venue configured end-to-end.** |
| **M3: Pilot-ready** | 7 | Compliance, load test and security checks pass. Soft launch at 1 friendly venue on a real Friday night. |
| **M4: Pilot live** | 8 | 5 venues live, analytics dashboard running, on-call rota active. |
| **Stage 2 kickoff** | 9+ | Gated on 3 owner requests for POS integration and ≥ 99% order accuracy. |

---

## 3. Sprint Breakdown

### Sprint 1 (Weeks 1–2): Foundation
- Terraform for ECS Fargate, RDS with RLS, S3/CloudFront, Cognito and SQS.
- Next.js and Fastify monorepo with GitHub Actions pipeline.
- Snapshot publish pipeline (JSON to S3 with versioning).
- US-1.1 QR/NFC routing and the "Pick your table" fallback.
- Design system with dark theme and 48px tap targets (US-1.3).
- **Demo:** scan a QR, see a static menu on a mid-range Android device.

### Sprint 2 (Weeks 3–4): The selling engine
- Menu browsing, allergen filters and age notice (US-1.4).
- Package engine with stepper, live price and one-tap add (US-2.1, 2.2). Rules are quantities per N and ⌈N/4⌉ platters, all configurable.
- Service worker, offline cart and the "Reconnecting…" state (US-1.2).
- Admin tool v0: venue, menu items and package CRUD.
- **Demo:** order a round for 6 people, offline and online.

### Sprint 3 (Weeks 5–6): Orders and operations
- Package swaps with price delta (US-2.3), and 86'd toggle with 60-second propagation (US-2.4).
- Server-side price and package revalidation, then Stripe checkout and webhooks.
- Order fan-out: SQS to SSE staff view, SES email and CloudPRNT printer worker, with DLQ alerts.
- Admin publish flow, venue templates and the 2-hour configuration playbook.
- Analytics events (no PII) and a basic owner report.
- **Demo:** intake sheet to live venue, timed.

### Sprint 4 (Weeks 7–8): Harden and launch
- Load test at 10× the expected Friday peak, plus a throttled-4G Lighthouse budget (< 2s).
- Accessibility and contrast pass, GDPR/CCPA cookie-free analytics review, and a Stripe PCI SAQ-A check.
- Soft launch (Week 7), bug bash, then staggered go-live of venues 2–5 (Week 8).
- Runbooks, dashboards and the pilot feedback loop.

---

## 4. Team Allocation

| Person | Role | Sprint focus |
|---|---|---|
| **Rafa Lindqvist** | Tech lead, backend | Orders module, SQS workers, Stripe, server-side validation, release management |
| **Tomás Reyes** | Frontend engineer | Guest PWA, package UI, offline behavior, performance budget |
| **Aiko Sato** | Full-stack and infra | Terraform, CI/CD, admin tool, staff view, printer integration |
| Milo Chen (PM) | Not in squad | Venue intake, acceptance criteria, pilot sales support |
| Ines Okafor (Architect) | About 4 hrs/week | Design reviews, snapshot and RLS sign-off |
| Design (contract) | Weeks 1–4 | Templates, dark-theme components, 3 venue skins |

---

## 5. Dependencies

| Dependency | Needed by | Owner | Fallback |
|---|---|---|---|
| Signed pilot venues and menus and pricing data | Week 3 | Milo / Ada | Use 2 friendly venues, delay venues 4–5 to Week 9 |
| Stripe account and alcohol-merchant approval | Week 4 | Rafa | Pay-at-table flag: order goes to staff, payment handled by venue |
| Star CloudPRNT printers at venues | Week 6 | Aiko | Email and staff-view only |
| NFC tags (NTAG213) and QR stickers printed | Week 7 | Milo | QR-only launch, NFC added later |
| Legal review of age and allergen copy | Week 5 | Ada | Ship conservative default copy |

---

## 6. Risks and Mitigations

| Risk | Likelihood / Impact | Mitigation |
|---|---|---|
| Scope creep from pilot owners ("just one custom feature") | High / High | Template-only rule. Requests go to the backlog, and Milo signs off on any exception. |
| Venue data arrives late or messy | High / Med | Standard intake sheet, and Aiko builds a CSV importer in Sprint 3. |
| Weak Wi-Fi breaks ordering | Med / High | Service worker, queued cart, and the strict snapshot pattern with no DB reads for guests. |
| Payment or alcohol-compliance delay | Med / High | The pay-at-table fallback above. Payment is a feature flag per venue. |
| Bus factor of 3 (illness on a Friday launch) | Med / High | Pair on the critical paths (orders and publish). Freeze deploys on Fridays after 12:00. |
| Friday-night incident during pilot | Med / Med | Alerts on DLQ depth and error rate, a rota rotating among the three engineers, and rollback via snapshot versioning. |
| Stage 2 POS uncertainty | Med / Low now | Keep the adapter interface in the orders module. Choose Square/Toast/Lightspeed from pilot data by Week 8. |

---

## 7. Launch Checklist

**Product and content**
- [ ] 5 venues configured with menus, packages, allergen tags and branding
- [ ] Owner walkthrough done and weekly uplift report scheduled

**Engineering**
- [ ] p75 load < 2s on throttled 4G, and the menu opens offline after the first visit
- [ ] The same URL opens from both QR and NFC on iOS and Android
- [ ] Price revalidation tested, so tampered client totals are rejected
- [ ] Order delivery to staff view, email and printer verified per venue, with DLQ alerting
- [ ] RLS tenant-isolation tests pass, and Cognito MFA is enforced for admins
- [ ] Rollback rehearsed (previous snapshot restored in < 5 minutes)

**Compliance**
- [ ] Age notice and allergen copy approved
- [ ] Analytics contain no PII, and no card data is stored (Stripe-hosted)

**Operations**
- [ ] On-call rota, runbooks and a venue support phone line ready
- [ ] Day-60 metrics dashboard live against the Brief targets (attach rate ≥ 30%, check uplift ≥ 12%)

---

## 8. Beyond Stage 1 (Directional)

- **Weeks 9–20, Stage 2:** POS adapters for the top two systems among pilots, reconciliation and order-accuracy monitoring. Requires 1 additional engineer.
- **Stage 3:** research only, with no hardware spend until a venue signs a paid pilot.