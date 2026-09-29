# MENUZ V3 CLAUDE

Clickable Stage 1 prototype of **Menuz Tableside**: a mobile web menu opened by QR/NFC that sells group packages ("The Round") and sends orders to a staff view.

Single file, no build step, no dependencies. Open `index.html` in a browser (best on a phone-sized window).

## What's inside
- Guest menu: dark, one-handed layout, allergen tags and filters, 18+/21+ notice
- The Round: 2-12 people, live price, swaps with price delta, one-tap add
- Cart, pay-at-table or simulated pay-now, server-side price re-check, idempotent submit, "Same again"
- Staff view: live tickets, status flow, sold-out list, simple insights
- Prototype tools: offline mode, failed payment, venue skins, bad table code, tampered-total test

Open the page in two tabs (Guest and Staff) to see orders sync between them.

## Not built yet
Admin tool (venue/package editor, publish and rollback, QR/NFC export), real backend (Stripe, queue, printer, email), real service-worker offline cache.

## Source briefs
Executive delegation brief, PRD, user stories, system architecture and execution roadmap for Menuz Tableside (Stage 1, 8-week pilot).
