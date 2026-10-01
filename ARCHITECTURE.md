# Architecture — Tiệm Của Mây

## Project boundary

This is a separate project from MIDORA. No MIDORA remote is configured. The approved design and supplied logo are retained under `../design-targets/`; no lighting domain or database artifacts are brought into this workspace.

## Initial layout

- Public UI: Next.js App Router, TypeScript strict mode, mobile-first responsive CSS.
- API (later batch): Node.js/TypeScript + Fastify, modular monolith.
- Data: separate local PostgreSQL database on port 5438 and a new migration history. Store VND prices as integers. Product image bytes live in persistent storage at `/data/uploads/products`; PostgreSQL stores generated media keys and metadata.
- UI prototype uses clearly labeled DEMO product data. Its source imagery is a design reference and must be replaced by separately managed catalog/banner assets before production.

## Owner gates

The approved Batch 3 scope permits guest checkout as a manual order request. Online payments, COD/bank-transfer semantics, shipping zones, returns/refunds, customer accounts, and external messaging integrations remain out of scope. Production shipping stays nullable until configured by the owner.

## Batch status

- Batch 0 workspace folders, approved design asset copies, local PostgreSQL Compose config, and a clean baseline migration: initialized.
- Batch 1 public homepage: visual prototype implemented; awaiting owner visual review.
- Batch 2 catalog: implemented and owner approved.
- Batch 3 anonymous cart, discount projection, manual order requests, centralized settings, inventory lifecycle, protected admin, and QA: implemented; ready for owner review.
- Batch 4 protected catalog administration, variants, product image lifecycle, inventory editing, persistent uploads, and regression QA: implemented; ready for owner review.

See `BATCH-3-REPORT.md` and `BATCH-4-REPORT.md` for scope, architecture, QA evidence, and screenshots.
