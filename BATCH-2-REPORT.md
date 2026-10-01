# Batch 2 — Fashion Catalog and Public Read API

**Status: READY FOR OWNER REVIEW**

## Project boundary and visual source

Batch 1 stays the approved visual source. The homepage layout, brand, pastel palette, hero, search styling, category row, product grid, card radii, typography, and bottom navigation were retained. Homepage content now comes from the public catalog API; category shortcuts and product links navigate to functioning routes. No MIDORA domain table, migration, product data, or remote was imported.

## Domain decisions

| Entity | Implementation |
| --- | --- |
| Category | Unique slug/name, nullable description and image/icon metadata, stable sort order, active flag, timestamps, explicit TEST/PRODUCTION provenance. Inactive categories are omitted. |
| Product | Unique slug, name/description/category, DRAFT/ACTIVE/INACTIVE/DISCONTINUED lifecycle, integer `base_price_vnd`, explicit `is_featured`/`is_new`, provenance, timestamps. |
| ProductVariant | Parent product, unique SKU, open text size, stable color code/name and optional display/hex, nullable integer override, non-negative integer stock, active flag. |
| ProductImage | Product, optional variant, URL/path, alt text, stable ordering, primary flag, timestamp. No upload/storage service. |
| Collection | Omitted: no approved collection data or current UI behavior needs it. |

### Price and inventory

Base and override prices are PostgreSQL integers in VND. The API returns integer effective variant prices, inheriting the product base price when no override exists. Variant stock is an integer constrained to zero or greater; availability is derived as `IN_STOCK` or `OUT_OF_STOCK`. No warehouse, reservation, or sales-derived best-seller model was added.

### TEST/prod separation

Both categories and products carry `PRODUCTION` or `TEST` provenance. API default `CATALOG_MODE=production` selects only production rows. `CATALOG_MODE=test` must be set explicitly to include fixtures, and the seed script rejects `NODE_ENV=production` or any non-test catalog mode. Public JSON never returns provenance. Migration history starts with `0000_initial.sql` (pgcrypto) and `0001_fashion_catalog.sql`; tested from a fresh isolated `tiem_cua_may_test` database.

## API and storefront

- `GET /api/v1/public/categories` returns active, eligible categories in deterministic order.
- `GET /api/v1/public/products` supports bounded `q`, `category`, `newOnly`, sort, page, and limit.
- `GET /api/v1/public/products/:slug` returns public product fields, active variants, effective prices, stock availability, and ordered public images.
- `GET /api/v1/public/categories/:slug/products` uses the same filtered listing model.
- Search covers product name, category name, and active variant SKU using PostgreSQL `ILIKE` with escaped query text; maximum query length 80.
- Sort: `newest`, `price_asc`, `price_desc`, `name`. Pagination maximum is 50 products per page.
- Zod validates query values and FE API response shape. PostgreSQL enforces slug/SKU uniqueness, lifecycle enums, non-negative prices/stock, and image URL/path shape. Errors expose safe public messages; CORS is allowlisted; request body and URL sizes are bounded; authorization/cookies are redacted from logs.
- Homepage “Mới về” uses `is_new=true`; active categories and product cards are API-backed. Search submits to `/products`; category shortcuts filter `/products`; product cards open detail pages. No fake product fallback is used when the API is unavailable.
- Product listing supports query, category, price/name/newest sorting, pagination, and a truthful empty/unavailable state. Detail shows ordered images and lets users select a color and size to see its effective price and stock state.
- Cart, accounts, persistent favorites, admin, checkout, orders, payment, promotion, and real shipping/returns policy are out of scope. Header cart, account, and favorite affordances do not fabricate counts or state. Demo product imagery had overlaid sales/favorite stickers removed from the TEST image assets.

## TEST fixtures

Five TEST-only fashion categories and six TEST products cover dresses, tops, pants, skirts, accessories, multiple sizes/colors, one out-of-stock variant, variant price override, and multi-image metadata. All test records have explicit TEST provenance; fixture copy identifies them as development data.

## Verification

- Backend unit validation: **3/3 passed**.
- PostgreSQL API/constraint integration suite A–O: **10/10 passed**, including active-only visibility, detail/variants/pricing, non-negative stock, duplicate SKU/slug, search, category filtering, sorting/pagination bounds, inactive product/variant hiding, and production exclusion of TEST-only records.
- Frontend API/schema and currency tests: **4/4 passed**.
- Real Playwright E2E against the running Fastify API and PostgreSQL fixture database: **passed**. Homepage products, category filter, text search, sort submit, detail images, color/size options, and override price were verified without API mocks.
- Responsive Playwright captures at 390×844 and 1440×900 for home/list/detail: no horizontal overflow and zero page errors. Captures: `qa/screenshots/batch2/`.
- Backend typecheck, lint, build: **passed**.
- Frontend typecheck, lint, tests (4/4), production build: **passed**.
- Production-mode frontend empty-catalog E2E: **passed**; no TEST product/category fallback appears.
- PostgreSQL 17 local compose service is available. Migrations ran from empty test DB; TEST seed completed.

Responsive Playwright screenshots:

- `qa/screenshots/batch2/homepage-mobile-390x844.png`
- `qa/screenshots/batch2/homepage-desktop-1440x900.png`
- `qa/screenshots/batch2/listing-mobile-390x844.png`
- `qa/screenshots/batch2/listing-desktop-1440x900.png`
- `qa/screenshots/batch2/detail-mobile-390x844.png`
- `qa/screenshots/batch2/detail-desktop-1440x900.png`

## Assumptions and limitations

- Initial categories/products/images remain TEST fixtures; Owner has not supplied production catalog content.
- Images are existing local demo product crops. Image upload and production media hosting are later work.
- Search is substring based and case-insensitive; no Elasticsearch, tokenization, or accent folding.
- Public APIs are read-only. No admin writes are exposed.
- Price and stock are current catalog snapshots; no reservations or cart availability promises exist.
- Account/favorites/cart persistence and commercial policies remain future owner decisions.

## Local repositories

FE, BE, DOC, and workspace changes are local commits only; there are no configured remotes and nothing was pushed.

- FE: `7f27ef2` — API-backed homepage, product listing/detail, tests, and demo imagery.
- BE: `73c422b` — clean fashion schema/migration, public API, fixture seed, and integration suite.
- Workspace: `212864b` — run instructions and six Playwright screenshots.
- DOC: this Batch 2 report commit.

No production catalog has been seeded. The development DB has migrations applied; the isolated test DB contains only explicit TEST fixture records. Cart/checkout/Batch 3 remain unstarted.
