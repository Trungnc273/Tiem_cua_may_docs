# Batch 3 — Commerce Order V1

**Status: BATCH 3 = READY FOR REVIEW**

Batch 3 implements the owner-authorized guest cart and manual order request flow. It does not start Batch 4.

## Delivery commits

- FE: `55acf29` — `feat: add guest cart and manual orders`.
- BE: `39b8530` — `feat: add transactional commerce order api`.
- DOC: this report is included in the Batch 3 documentation commit.
- Workspace README and screenshot evidence: `f0d152d`.

## Commerce behavior

- A product discount is a product-level integer from 0 through 100. A variant price override takes precedence over the product base price; otherwise the base price applies. The backend publishes `originalPriceVnd`, `salePriceVnd`, `discountPercent`, and `hasDiscount` projections for catalog cards and variant detail.
- Discount rounding is deterministic, nearest whole VND with half values rounded up: `floor((originalPriceVnd * (100 - discountPercent) + 50) / 100)`. Prices and totals remain integer VND. The client never calculates an authoritative total.
- The anonymous cart persists in PostgreSQL across reloads and navigation. The browser receives an opaque 256-bit `tcm_cart` HttpOnly, SameSite=Lax cookie (Secure in production); only its SHA-256 digest is stored. The cart session is scoped to `PRODUCTION` or `TEST` so changing API mode cannot carry TEST products into production. The cart stores variant IDs and quantities, and the backend resolves current availability, stock, discounts, and prices.
- The customer form asks for name, Vietnam phone number, delivery address, and an optional note. It does not ask for an email or account. The phone validator accepts unambiguous 10/11-digit local numbers and `+84` numbers that normalize to a valid 10-digit local number; ambiguous formats are rejected.
- Shipping is one centralized default amount in `store_settings`. The production record starts `NULL`, with no hardcoded production fee. An empty setting is shown to customers and blocks order submission. The isolated TEST seed explicitly uses 25,000 VND.
- The API accepts only a UUID idempotency key, customer fields, and note from the browser. It computes item prices, shipping, subtotal, and total. Successful order submission returns only the human-readable order code and initial status; no public order readback endpoint exists.
- Each order stores immutable customer and total fields. Each order item snapshots product name, variant SKU/color/size, image, original unit price, discount, sale unit price, quantity, and line total. PostgreSQL triggers reject snapshot edits; price/settings changes do not alter existing order values.
- Order codes are `TCM-` plus ten cryptographically random uppercase hex characters and have a unique database constraint. Order idempotency is unique per cart session and key, checks that retries carry the same request payload, and serializes submission on the cart session.
- Order placement locks variant rows and decrements stock within the same transaction as order creation. Database checks prevent negative stock; concurrent orders for the last item serialize, so only one succeeds. Cancelling `NEW` or `CONFIRMED` restores stock transactionally. Cancellation is rejected after `SHIPPING`.
- Allowed transitions are `NEW → CONFIRMED → SHIPPING → COMPLETED`; cancellation is available from `NEW` and `CONFIRMED` only. Completed and cancelled orders are terminal.

## Routes and admin

| Area | Routes | Behavior |
|---|---|---|
| Public store | `GET /api/v1/public/store-settings` | Contact phone and Messenger URL from centralized settings; shipping readiness only |
| Cart | `GET /api/v1/public/cart`, `POST /api/v1/public/cart/items`, `PATCH/DELETE /api/v1/public/cart/items/:itemId` | Cookie-backed cart, stock-bounded quantities, current price projection |
| Order | `POST /api/v1/public/orders` | Manual request; server totals, idempotency, stock transaction; no payment or public readback |
| Admin auth | `POST /api/v1/admin/auth/login`, `POST /logout`, `GET /me` | Scrypt password hash, opaque HttpOnly cookie, 12-hour expiry, logout revocation, login throttling, no public registration |
| Admin orders | `GET /api/v1/admin/orders`, `GET /:orderId`, `PATCH /:orderId/status` | Bounded list/filter, last four phone digits in list, full PII only in authenticated detail, guarded status changes |
| Admin products | `GET/PATCH /api/v1/admin/products` | Edit base price, product discount, and active status |
| Admin settings | `GET/PATCH /api/v1/admin/settings` | Edit phone, allowlisted Facebook Messenger URL, and nullable default shipping fee |

Admin screens are `/admin/login`, `/admin/orders`, `/admin/orders/[orderId]`, and `/admin/products`. The first account is provisioned with `pnpm admin:create`; `ADMIN_EMAIL` and `ADMIN_PASSWORD` are read from the protected process environment, and no credential is printed or built in. Admin pages and responses use `Cache-Control: no-store`.

Admin actions append audit events for successful login/logout, order status changes, product price/discount changes, and store settings changes. Audit metadata records only changed fields/statuses and numeric values; it excludes credentials, phone, address, and customer note. Audit events are append-only.

The product detail offers phone contact via `tel:` and the approved Messenger page in a new tab with `noopener noreferrer`. No message or call is triggered automatically. External Messenger URLs are constrained to HTTPS Facebook URLs.

## Security and migration notes

- Migrations `0002_commerce.sql` and `0003_snapshot_guards.sql` are additive to the approved Batch 2 schema. No database reset is required.
- New tables cover store settings, cart sessions/items, orders/items, admin users/sessions, and audit events. PostgreSQL constraints enforce bounded discount/quantity values, nonnegative money/stock, unique order/idempotency keys, total arithmetic, and snapshot rules.
- Cookie values and password hashes are never placed in browser storage. Logs do not include request bodies; the generic API error logger records only the error name. State-changing browser requests enforce the configured Origin allowlist, and credentialed CORS is limited to configured storefront origins.
- Production catalog mode rejects TEST variants from cart APIs even if a client submits a known TEST variant UUID. A mode-specific cart cookie is issued and persisted separately.
- No payment gateway, COD/bank-transfer label, shipping-zone pricing, cancellation-after-shipping, variant-specific discount, customer account, or external messaging integration was added.

## UI and visual QA

- The approved homepage structure remains in place. Its cart affordance is now functional and its returns copy is replaced by neutral store-information text; there is no free-shipping claim.
- Product detail shows the original price crossed out only when discounted, the actual sale price, and the percent badge. It offers variant selection, stock state, add-to-cart, phone, and Messenger actions.
- The `/gio-hang` screen supports quantity changes and removal, shows current prices, configured shipping, and server-derived totals. It blocks submission for unavailable stock or unconfigured shipping.
- Successful submission shows the order code and explains that the store will contact the customer to confirm. It does not offer or promise a payment method, and does not display sensitive delivery data.
- Admin list masks phone numbers to the last four digits. Full contact details and order snapshots appear only in authenticated detail.
- Screenshots: product detail, cart/order form, order success, and unconfigured shipping at 390×844; admin settings/orders/order detail at 1440×900; homepage and listing regression captures at both sizes. Exact artifact sizes appear below.

## Verification

Final local verification for the owner-authorized Batch 3 pass:

- BE `pnpm typecheck`, `pnpm lint`, `pnpm test` (3/3), `pnpm test:integration` (15/15), and `pnpm build`: passed.
- FE `pnpm test` (4/4), `pnpm typecheck`, `pnpm lint`, and `pnpm build`: passed.
- Existing catalog Playwright regression: passed (four API-backed homepage products, category/search/sort/variant price, no horizontal overflow, zero browser errors).
- Commerce Playwright journey: passed with order `TCM-038F009D1D`; admin sets a 20% product discount and 25,000 VND shipping; the browser confirms the sale price, adds a variant to cart, submits a guest order, and views the order in admin. A second pass removes the TEST shipping fee, verifies the customer warning and disabled submit button, then restores the TEST fee.
- Integration coverage includes discount calculation, stock and quantity validation, missing shipping configuration, request idempotency, immutable snapshots, cancellation/restoration, concurrent last-stock attempts, status transition guards, admin authorization, Origin allowlisting, protected PII, and TEST/PRODUCTION isolation.
- Playwright uses the real API and PostgreSQL-backed TEST catalog without mocked API calls. Screenshots were checked for overflow at 390×844 and 1440×900; the browser reported zero page errors.

Screenshot artifacts and byte sizes:

| Artifact | Bytes |
|---|---:|
| `product-detail-mobile-390x844.png` | 256,787 |
| `cart-order-form-mobile-390x844.png` | 61,700 |
| `order-success-mobile-390x844.png` | 25,748 |
| `shipping-unconfigured-mobile-390x844.png` | 56,354 |
| `admin-products-settings-desktop-1440x900.png` | 102,177 |
| `admin-orders-desktop-1440x900.png` | 45,584 |
| `admin-order-detail-desktop-1440x900.png` | 50,713 |
| `homepage-regression-mobile-390x844.png` | 280,240 |
| `homepage-regression-desktop-1440x900.png` | 1,010,970 |
| `listing-regression-mobile-390x844.png` | 210,049 |
| `listing-regression-desktop-1440x900.png` | 924,550 |

## Assumptions and remaining owner decisions

- The owner approved a single default shipping amount with no location-based calculation. Until a production administrator sets it, production checkout remains blocked.
- The owner approved a manual order confirmation flow; no payment/COD/bank-transfer semantics are represented.
- The owner approved product-level discounts and cancellation only before shipping.
- No material scope blocker remains. Batch 3 is ready for owner review; do not start the next batch automatically.
