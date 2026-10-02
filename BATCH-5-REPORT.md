# Batch 5 — Production Readiness and Local Public Staging

Date: 2026-10-02  
Project: Tiệm Của Mây  
Status: **Ready for owner phone/external-network review. Stop here before VPS or production deployment.**

## Accepted baseline and Batch 5 branch

Batch 4 `main` baseline:

| Repository | Final Batch 4 `main` |
| --- | --- |
| FE | `ae5e5913529b42315444024235a254cc0ae2d936` |
| BE | `b38592b6487c70964ed5124691db259262235ccd` |
| DOC | `22af6154575c4e117be9dae91a3830aa576decbd` |
| Workspace / QA | `5495cb5` |

Batch 5 local branch: `batch5/production-readiness-local-staging`.

| Repository | Batch 5 commit |
| --- | --- |
| FE | `c8c516dc66f46b2fec22d7143a57549b17c11a51` |
| BE | `f40f92693f882df0c2499f000e04573993e1bf63` |
| DOC | This report and runbook commit |
| Workspace / QA | Production/staging Compose, backup tools, and final screenshots committed on the Batch 5 branch |

The Batch 5 commits are local owner-review candidates. The approved Batch 4 `main` history was pushed normally in Phase A. Batch 5 has not been deployed to a VPS.

## V1 behavior and production gates

The V1 business model remains guest order requests, manual order confirmation, one configurable shipping fee, product-level discounts, per-variant stock, and phone/Messenger consultation. No payment, customer accounts, carrier integration, or other commerce feature was added.

The shop settings use the owner-approved phone `0876146498` and Messenger URL `https://www.facebook.com/tiemcuamay04`. They are sourced from validated store settings rather than FE constants. The HTTPS browser run read and checked both values.

Production shipping remains unset because the owner has not chosen an amount. Checkout stays blocked until Admin configures it. This is the **SHIPPING CONFIG GATE**. The isolated staging fixture uses 25,000 VND only as TEST data; the acceptance run also removed that TEST fee and confirmed that checkout became disabled before restoring the fixture.

No real product names, prices, variants, stock, descriptions, or photos were supplied for production entry. Staging uses TEST catalog data and a synthetic upload fixture, all marked TEST. Owner/shop-supplied catalog details and photos are needed before production catalog entry. No AI or stock imagery is presented as a real item.

Public copy was reviewed for unsupported free shipping, return, online payment security, or round-the-clock support promises; the owner has not approved those policies.

## Compose, runtime, and environment

- `docker-compose.staging.yml` runs production-mode API/web containers with a dedicated TEST PostgreSQL database. It publishes PostgreSQL only on loopback port 55439; the API has no host port; the web is loopback-only on port 3180.
- Same-origin Next.js `/api/*` rewrites route HTTPS browser calls to the private API service. The flow is public HTTPS tunnel → local web → private API → private PostgreSQL and product-upload volume.
- Persistent staging volumes are `tcm_batch5_stage_postgres` and `tcm_batch5_stage_product_uploads`.
- `docker-compose.production.yml` is a separate runtime template. It accepts release-pinned API/web image references; the DB and API have no published host ports; only web binds to loopback for a future approved tunnel ingress. It has health checks, restart policies, bounded logs, persistent database/media volumes, no hot reload, and no fixture seed service.
- `.env.production.example` contains placeholders only. Production startup requires `TCM_ENVIRONMENT=production`, `CATALOG_MODE=production`, a valid HTTPS public origin and exact CORS allowlist, a DB identity/password, and pinned image references. The production FE image must be built with `API_INTERNAL_URL=http://api:4000` so the server-side same-origin rewrite targets the private API.
- Migration runs explicitly through the `migration` Compose profile and applies only pending versioned migrations. API boot never resets or seeds a database. `/ready` requires `0004_product_admin_uploads.sql`.
- `scripts/create-admin.ts` is compiled into the production API image for the operator CLI. Production Admin creation requires production provenance and injected credentials; TEST creation requires the exact local TEST database allowlist. TEST credentials are not committed or auto-created in production.

## Security, privacy, and storage

- Admin and cart cookies are HttpOnly, SameSite=Lax, and Secure in production. Production FE/API configuration fails fast on missing or invalid required environment.
- CORS accepts exact origins without wildcards. State-changing commerce and catalog-admin routes require an approved matching Origin.
- API request logs omit bodies and redact Cookie and Authorization headers. Order PII is absent from URLs and browser storage; Admin detail is access-controlled, and no public order lookup exists. Integration tests cover unauthorized order access and protected Admin PII.
- Image upload validates supported image bytes, generated storage keys, dimensions, size limits, and resolved storage paths. Media is served from the persistent volume with safe content type, `nosniff`, sandbox CSP, and immutable caching; filesystem paths are not returned.
- Docker logs rotate at 10 MiB with three files retained. Product files are capped at 8 MiB each. Upload-volume and paired-backup growth should be monitored.
- Secret scan covered the workspace and FE/BE/DOC repositories for private-key headers, common GitHub/AWS token shapes, tunnel-token names, and literal DB/Admin password assignments. No committed matches were found. `.env.staging.local`, the generated TEST admin credentials, tunnel logs, and backups remain ignored local files.
- `pnpm audit --prod` reports **No known vulnerabilities found** for both FE and BE after patching Next.js to `16.3.6` and Fastify to `5.12.5`.

## Staging E2E and screenshots

The final Playwright run used the temporary HTTPS Quick Tunnel URL below, an isolated TEST Admin, the patched production images, and no real customer PII. It passed with 0 browser errors and no horizontal overflow at 390×844.

Verified flow: Admin login/settings; category and product creation; discount and variant stock; synthetic image upload; public home/catalog/product and contact links; color/size selection; guest cart/order; shipping total; Admin order detail/status; immutable old order price after a later discount edit; stock 2→1→2 after eligible cancellation; blocked checkout without configured shipping; and logout.

The TEST checkout total was 276,250 VND sale subtotal + 25,000 VND TEST shipping = 301,250 VND. Changing the current product from 15% to 20% did not change the saved order snapshot.

Screenshots in `../qa/batch5/`:

- Admin: `admin-settings-desktop-1440x900.png`, `admin-categories-desktop-1440x900.png`, `admin-products-desktop-1440x900.png`, `admin-product-editor-desktop-1440x900.png`, `admin-orders-desktop-1440x900.png`, `admin-order-detail-desktop-1440x900.png`.
- Mobile at 390×844: `home-mobile-390x844.png`, `catalog-mobile-390x844.png`, `product-detail-mobile-390x844.png`, `cart-mobile-390x844.png`, `order-success-mobile-390x844.png`.

## Backup and restore evidence

The final TEST backup set is `backups/stage-20261002-085552/`. It contains a PostgreSQL custom-format dump and `product-uploads.tar.gz`; SHA-256 manifest:

```text
477b0129e1ec729d6132dd116205506d49cf728e02c60238a0b76b93a86a66da  product-uploads.tar.gz
0b653b8a1a68426301e22395d8a8f9f06ce5136b9518308596b5434051aa296f  tiem-cua-may-20261002-085552.dump
```

Restore drill passed from that set into new isolated resources, leaving the source staging DB/volume unchanged:

- Restore DB: `tiem_cua_may_restore_stage_20261002085614_stage`
- Restore upload volume: `tcm_restore_uploads_20261002085614`
- Verified readiness, TEST product, variants, uploaded image delivery, Admin login, one order, and store settings.

See `LOCAL-PUBLIC-STAGING.md` for the operator commands and production backup requirements. Production release must take a protected off-host backup pair and complete a separate isolated restore drill before real data cutover.

## Automated checks

- FE: lint, typecheck, 4/4 unit tests, production build — passed on Next.js 16.3.6.
- BE: lint, typecheck, 4/4 unit tests, production build — passed on Fastify 5.12.5.
- PostgreSQL integration: catalog 10/10, commerce 5/5, product-admin/media 1/1 — passed against the isolated staging database.
- Docker Compose staging and production templates: `config -q` — passed.
- Public HTTPS Playwright journey: passed; 0 browser errors; no 390×844 horizontal overflow.
- Upload persistence: passed after API restart, API image rebuild/recreate, and web image rebuild/recreate.
- Paired PostgreSQL/upload-volume backup and isolated restore drill: passed.
- Dependency audit and secret-pattern scan: passed.
- `git diff --check`: clean before commit.

## Owner review gate

Temporary HTTPS staging URL: `https://usr-yellow-attempt-band.trycloudflare.com`  
TEST Admin email: `owner-review-b5@tiemcuamay.test`  
TEST Admin password: delivered directly to the owner in the completion response; not stored in Git.

Keep the local stack and Quick Tunnel running for review from a real phone over 4G/5G or another external network. The temporary hostname expires when the tunnel process stops. VPS hostname/IP, SSH access, deployment path, production domain, Cloudflare production tunnel details, and any registry credentials are not supplied. The production templates are prepared, but no VPS/domain deployment is authorized at this stage.

**Stop here for owner external-device review. Do not begin Batch 6.**
