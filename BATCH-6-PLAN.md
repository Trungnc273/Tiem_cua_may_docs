# Batch 6 — Province estimates and manual shipping confirmation

Owner finalized Batch 6 v1 on 2026-10-03. This replaces the earlier distance/map proposal. Batch 6 removes maps, GPS, geocoding, road routing, Mapbox/Google Maps, and distance tiers. No map-provider account or carrier API credentials are needed.

## Customer flow

- Customer enters name, phone, province/city, detail address, and optional note.
- The province selector uses the 34-unit national list published for the 2025 administrative changes: [Government list of 34 provincial-level administrative units](https://xaydungchinhsach.chinhphu.vn/chi-tiet-34-don-vi-hanh-chinh-cap-tinh-tu-12-6-2025-119250612141845533.htm).
- Checkout displays an estimated minimum–maximum shipping range when available, plus the estimated total range. If there is no active province or fallback rule, the order can still be submitted and the customer is told staff will confirm shipping.
- Public pages do not show package weight, map pins, coordinates, or an exact payable total before staff confirms shipping.
- The order snapshots province and the estimate rule/range used at submission. Later rule changes do not alter historical orders.

## Admin shipping workflow

- Admin manages one active estimate rule per province and one active fallback rule. Prices are integer VND with `0 <= min <= max`.
- Staff enters the carrier (GHTK, J&T Express, Viettel Post, or custom), actual shipping fee, and optional tracking number on the order.
- The backend sets shipping to `CONFIRMED` and computes the exact total from the immutable item subtotal plus the entered fee. Client totals are ignored.
- `NEW → CONFIRMED` is blocked until the carrier and final fee are present.
- Carrier quote APIs are future work; no carrier API is called in Batch 6.

## Brevo notifications

- Use Brevo REST with sender `midoradesign@gmail.com` and approved recipient `ntvippro24@gmail.com`.
- Order commit is independent of email delivery. Keep the outbox and bounded retry worker. Email includes the province, address, item snapshots, discounts, subtotal, estimate range, and a clear notice that shipping is not final.
- Credentials remain server secrets. A production smoke email must be non-PII and must not reveal credentials.

## Database migration policy

- Batch 6 feature migration is `0006_distance_shipping_brevo_outbox.sql`; owner requires checking production `schema_migrations` before deciding whether it can be revised.
- `0007_shipping_estimates.sql` is currently additive and removes the abandoned distance schema while introducing province estimates and nullable pre-confirmation shipping totals. The original migration is preserved until production application state is verified; never rewrite an applied migration.
- Existing Batch 1–5 order/product data must be preserved. Migration state and production database backup/checksum must be confirmed before deployment.

## Test and release gates

- PostgreSQL integration tests require an isolated, local database explicitly named TEST/staging and will fail fast when that target is absent. Production is never a test target.
- Required integration coverage: province estimate, fallback, no-rule checkout, immutable estimate snapshot, nullable initial final fee/total, Admin carrier/final fee, server-side total, status guard, negative fee and inverted range rejection, and ignored client-supplied totals.
- Required email coverage: persisted order/outbox, successful send, failure does not roll back the order, bounded retry, and HTML escaping.
- Verify FE mobile checkout at 390×844, public root/www/media behavior, Admin login/order operations, R2, and that SUMFLOW remains unchanged.
- Back up the production database before migration. Build the exact approved Batch 6 SHA and deploy only Tiệm Của Mây services. Do not restart or modify SUMFLOW.

## Current evidence

- Approved Batch 5 source SHAs: FE `34daebef96a582240e726ba511dc9b1e16468ae6`, BE `abb8a7e89a0d6b6171f7e766f95ca470f8bed7f7`, DOC `6a0af54bb9c01b95f9cfe7453ac4c59045d5b08c`.
- Current checked-out heads before Batch 6 commits: FE `69c098f12ee9a17715574c1795e2be047923d36a`, BE `abb8a7e89a0d6b6171f7e766f95ca470f8bed7f7`, DOC `6a0af54bb9c01b95f9cfe7453ac4c59045d5b08c`. All three repositories are on `feat/distance-shipping-brevo-notifications`.
- Production migration ledger was queried read-only on 2026-10-03. Production is at `0005_r2_product_media.sql`; `0006_distance_shipping_brevo_outbox.sql` has not been applied. Keep the unapplied migration history intact and use the checked forward migration `0007_shipping_estimates.sql`; never rewrite applied history.
- A dedicated VPS TEST PostgreSQL (`tcm-batch6-test-postgres`, database `tiem_cua_may_test`, network and volume `tcm-batch6-test-*`) was created with a root-only TEST credential file. Production and SUMFLOW databases were not used by tests.
- FE and BE typecheck, lint, unit tests, and production builds pass. Unit totals: FE 4/4, BE 15/15. The complete PostgreSQL integration suite passes 17/17 with zero skipped tests.
- Real Playwright passed against the TEST API/database at 390×844 and 1440×900. It covered storefront, product detail, cart, estimate and no-estimate checkout, order creation, admin login/detail, carrier/final fee/tracking, and `NEW → CONFIRMED`. Result: no horizontal overflow, console errors, failed requests, cross-origin API calls, insecure external assets, or HTTP 5xx; `/health` and `/ready` returned 200.
- Browser evidence is in `qa/screenshots/batch6/`. The confirmed order screenshot shows carrier, tracking number, final shipping fee, and server-calculated total.
- No real Brevo email has been sent. The TEST API ran without a Brevo key; unit/integration tests used a fake adapter and exercised outbox success, retry, and persistence.
- Batch 6 changes remain uncommitted and unpushed; no Batch 6 release SHA exists yet. Production database backup, production Admin bootstrap, production Brevo secret validation, migration application, service deployment, www policy, public/R2 smoke, and final protected-service verification remain pending. Do not report READY FOR OWNER REVIEW until each authorized production gate has evidence.
