# System Architecture

_Department: Product & Tech · Agent: Ines Okafor · 9/29/2026, 6:57:15 PM_

---

# System Architecture: Menuz Tableside

**Owner:** Ines Okafor, System Architect
**Scope:** Stage 1 in detail; Stage 2 and 3 as extension points
**Principle:** Boring technology, one deployable backend, and static-first delivery so the menu loads fast on bad networks.

---

## 1. Stack and Rationale

| Layer | Choice | Why |
|---|---|---|
| Guest menu | Next.js (React, TypeScript), PWA with service worker | Static generation gives <2s loads on 4G. The service worker caches the menu for spotty Wi-Fi. No app download. |
| Staff view and admin | Same Next.js codebase, authenticated routes | One repo and one skill set for a 3-person squad. |
| API | Node.js 20 + Fastify (TypeScript), modular monolith | Simple to run. Modules (menu, orders, billing, integrations) can be split out later if needed. |
| Database | PostgreSQL 15 (AWS RDS, Multi-AZ) | Relational fit for menus and orders. Row-level security for tenant isolation. |
| Cache / rate limits | Redis (ElastiCache, smallest node) | Rate limiting and short-lived table sessions only. It is never the source of truth. |
| Queue | Amazon SQS | Order fan-out to email, print and (Stage 2) POS adapters, with retries and a dead-letter queue. |
| Hosting | AWS ECS Fargate, CloudFront + S3, Cloudflare WAF/DNS | Managed and no servers to patch. |
| Auth (staff/admin) | Amazon Cognito with MFA for Menuz admins | No custom password handling. |
| IaC / CI | Terraform, GitHub Actions | Reproducible staging and production. |

**Key decision: published menu snapshots.** Admin edits go into a draft. "Publish" renders a versioned JSON snapshot to S3/CloudFront, and guests read only that snapshot. Menu reads never hit the database, so a Friday-night spike cannot slow down ordering. Price and package rules are re-validated server-side when an order is placed.

---

## 2. Component Diagram

```
 Guest phone (QR scan / NFC tap -> same URL)
 https://menuz.app/v/{venue}/t/{tableCode}
            |
            v
   +-------------------+
   | Cloudflare WAF/DNS|
   +----+---------+----+
        |         |
   static/JSON   /api/*
        |         |
 +------v----+ +--v-----------------------+       +------------------+
 | CloudFront| | ALB -> ECS Fargate       |<----->| Redis            |
 | + S3      | | Menuz API (Fastify)      |       | (rate limit,     |
 | menu      | |  modules:                |       |  table sessions) |
 | snapshots | |  menu | packages | orders|       +------------------+
 | + PWA     | |  payments | analytics    |
 +-----------+ |  admin | integrations    |       +------------------+
      ^        +--+-------+---------+-----+------>| PostgreSQL (RDS) |
      | publish   |       |         |             | RLS by venue_id  |
      |           |       v         v             +------------------+
      |           |  +--------+  +--------------+
      +-----------+  |  SQS   |  | Stripe       |
                     | orders |  | (payments,   |
   Staff PWA <--SSE--+---+----+  |  webhooks)   |
   (live orders)         |       +--------------+
                         v
        +----------------+-----------------+
        | Workers (ECS): email (SES),      |
        | printer (Star CloudPRNT poll),   |
        | Stage 2: POS adapters            |
        +----------------------------------+
   Admin tool (Menuz staff, Cognito + MFA) -> API
```

---

## 3. Data Model (Postgres)

Every tenant table carries `venue_id`, enforced by row-level security.

| Table | Key fields |
|---|---|
| `venues` | id, slug, name, brand_theme (JSON: colors, logo, fonts), timezone, currency, tax_rate, alcohol_notice_text, status |
| `venue_users` | id, venue_id, cognito_sub, role (`menuz_admin`, `owner`, `staff`) |
| `tables` | id, venue_id, label, table_code (unique short code), active |
| `menu_templates` | id, name, layout_config (JSON) |
| `menus` | id, venue_id, template_id, version, status (`draft`/`published`), snapshot_url, published_at |
| `categories` | id, menu_id, name, sort |
| `items` | id, venue_id, category_id, name, description, price_cents, image_url, is_alcohol, available, sort |
| `item_allergens` | item_id, allergen_code (EU 14 list) |
| `packages` | id, menu_id, name, tagline, min_guests, max_guests, discount_pct, hero_image, active |
| `package_components` | id, package_id, item_id, qty_mode (`per_guest` / `per_group_chunk`), chunk_size, qty |
| `orders` | id, venue_id, table_id, idempotency_key (unique), status (`received`/`accepted`/`void`), guest_count, subtotal_cents, tax_cents, total_cents, payment_mode (`pay_at_table`/`online`), menu_version, created_at |
| `order_lines` | id, order_id, item_id, package_id (nullable), name_snapshot, qty, unit_price_cents, notes |
| `payments` | id, order_id, stripe_payment_intent_id, status, amount_cents |
| `outbox` | id, order_id, channel (`email`/`print`/`pos`), status, attempts, last_error |
| `events` | id, venue_id, anon_session_id, type (`scan`, `package_view`, `order_start`, `order_placed`), props (JSON), ts |
| `integrations` *(Stage 2)* | id, venue_id, provider, external_location_id, encrypted_credentials, item_mapping (JSON) |

**Package pricing rule:** `per_guest` quantity = N × qty. `per_group_chunk` quantity = ceil(N / chunk_size) × qty. Example: "The Round" is beer per_guest, chips per_group_chunk (chunk 4), and a platter per_group_chunk (chunk 4). The server computes the price and the client only previews it.

---

## 4. API Surface (REST/JSON, versioned `/api/v1`)

**Guest (unauthenticated, rate limited)**
- `GET /v/{slug}/t/{code}`: resolves the table and returns a short-lived session token.
- `POST /sessions`: creates an anonymous session.
- `POST /quotes`: takes package ID and guest count, returns a server-calculated price.
- `POST /orders`: requires an `Idempotency-Key` header, and returns the order ID and status.
- `GET /orders/{id}`: order status.
- `POST /payments/intent`: creates a Stripe PaymentIntent.
- `POST /events`: batched anonymous analytics.

**Staff**
- `GET /staff/orders/stream`: SSE stream of live orders.
- `PATCH /staff/orders/{id}`: accept or void an order.

**Admin (Menuz staff and owners)**
- CRUD on `/admin/venues`, `/menus`, `/items`, `/packages`, `/tables`.
- `POST /admin/menus/{id}/publish`.
- `GET /admin/venues/{id}/report`: weekly uplift report.
- `POST /admin/tables/qr-export`: QR and NFC URL sheet.

**Webhooks:** `POST /webhooks/stripe` (signature verified).

---

## 5. Security and Compliance

- **Transport:** TLS 1.2+ with HSTS, and Cloudflare WAF with bot rules.
- **Tenant isolation:** Postgres RLS on `venue_id`, tested in CI.
- **Access control:** RBAC by role. MFA is mandatory for `menuz_admin`. Secrets live in AWS Secrets Manager, and encryption at rest is on for RDS and S3.
- **Abuse control:** Table codes are not secrets. Orders need a valid session, are rate limited per IP and table, and staff can void them. A per-table cap and duplicate-order detection prevent prank orders.
- **Payments:** Stripe Payment Element. Card data never touches Menuz (PCI SAQ-A). Pay-at-table is the fallback.
- **Privacy:** No login, no cookies beyond a rotating anonymous session ID, and no IP storage in `events`. Analytics are first-party only, so a consent banner is not needed. There is a 13-month retention limit on events.
- **Alcohol and allergens:** The venue-configured alcohol notice appears before alcohol packages can be added, and allergen tags show on every item and package.
- **Auditing:** Admin changes are logged and menu versions are immutable.

---

## 6. Scalability and Resilience

- **Targets:** 500 venues, a peak of about 200 API requests/second, and 99.9% availability.
- **Reads:** Guest traffic is served from CloudFront. The menu snapshot is under 100 KB gzipped, with images in WebP and a responsive `srcset`.
- **Writes:** The API is stateless and autoscales on CPU (2–10 tasks). Postgres handles this load on one primary, with a read replica added if reporting grows.
- **Weak network:** The client queues orders locally and retries with the same idempotency key. If SSE drops, the staff view falls back to 5-second polling.
- **Delivery fallbacks:** If SSE, print or email fails, the order stays in the outbox and retries. After 3 failed attempts it triggers an alert and shows a red banner in the staff view.
- **Observability:** Sentry, CloudWatch alarms, uptime checks, and a dashboard for order latency and outbox failures.

---

## 7. Third-Party Services

| Service | Purpose | Stage |
|---|---|---|
| Stripe | Payments | 1 |
| AWS SES | Order email fallback | 1 |
| Star CloudPRNT / Epson Server Direct Print | Kitchen and bar tickets without on-site software | 1 |
| Cloudflare, CloudFront | WAF, CDN | 1 |
| Cognito, Sentry | Auth, error tracking | 1 |
| Square / Toast / Lightspeed APIs | POS adapters (the two most common among pilots) | 2 |
| Web-based scene player over WebSocket | Table-synced video (research only) | 3 |

---

## 8. Extension Points

- **Stage 2:** Workers implement a `PosAdapter` interface (`mapItems`, `submitOrder`, `getStatus`). They read from the existing `outbox`, so order creation does not change. The 99% accuracy gate is measured as POS-confirmed orders divided by orders placed.
- **Stage 3:** A `scenes` table and a per-table WebSocket channel can be added without touching Stage 1 services. No hardware decision is made until a venue commits to a paid pilot.

## 9. Open Risks

| Risk | Mitigation |
|---|---|
| Fake or prank orders from public table URLs | Sessions, rate limits, staff void, optional per-table rotating code |
| Thermal printer variability across venues | Standardize on Star CloudPRNT for pilots, with email fallback |
| 48-hour setup promise | Template library, CSV item import, and QR/NFC export from the admin tool |