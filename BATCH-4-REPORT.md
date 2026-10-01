# Batch 4 — Catalog administration, variants, images, and inventory

**Status: READY FOR OWNER REVIEW.** This report closes Batch 4 only. Batch 5 has not started, and no deployment was performed.

## Delivered

- Added authenticated category and product administration: create and edit categories, activate or hide them, create product drafts, edit catalog fields, filter and page through products, and publish only when the product has an active category, active variant, and image.
- Added variant administration for unique SKU, free-text size and color, optional color swatch and price override, active state, and stock quantity. Existing order and inventory lifecycle semantics remain in force.
- Added product image upload, primary selection, ordering, optional variant association, and removal. Accepted formats are JPEG, PNG, and WebP, up to 8 MiB; the API checks file signatures and image dimensions, generates storage names, and serves only allowed generated keys. Product snapshots retain image references used by existing orders.
- Added a persistent Docker volume mounted at `/data/uploads/products` and documented the requirement to back up that volume with the database.
- Kept the existing order, cart, discounts, contact settings, authentication, and audit contracts. Store settings remain on their own admin page. Public routes reserve space for the fixed mobile navigation.

## Verification

- API: typecheck, lint, unit tests, and build passed. PostgreSQL integration passed sequentially: 16/16 tests (catalog 10/10, commerce 5/5, Batch 4 admin 1/1), including category and SKU filtering, supported image formats, rejected SVG and oversized uploads, unauthorized access, path traversal, stale writes and audit records, publish validation, stock `2 → 1 → 2`, and order snapshot retention after catalog edits and image removal.
- Web: typecheck, lint, unit tests, and production build passed. Two Playwright journeys passed against the isolated TEST database and local filesystem: Batch 4 product create, variant and image management, publish, cart, and guest order; plus the Batch 3 commerce, settings, and order-admin regression. Both reported zero browser errors and no horizontal overflow.
- Docker Compose configuration validation passed. QA used the local test database and local API/web processes only; no production database or VPS was contacted.
- Browser evidence at 1440×900 and 390×844 is in `../qa/batch4/`: admin product list/editor and public home, listing, detail, cart, and order confirmation. The E2E journey also asserts mobile bottom-navigation space on every public route.

## Boundaries

- Manual order requests remain the checkout model. Online payment, shipping integrations, refunds, and customer accounts remain outside Batch 4.
- Products, categories, and variants are deactivated rather than hard-deleted. Historical order snapshots remain stable.
- Product photography is owner supplied. No generated product images were added.
- Review checkpoint: **BATCH 4 = READY FOR REVIEW**.
