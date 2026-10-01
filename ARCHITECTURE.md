# Architecture — Tiệm Của Mây

## Project boundary

This is a separate project from MIDORA. No MIDORA remote is configured. The approved design and supplied logo are retained under `../design-targets/`; no lighting domain or database artifacts are brought into this workspace.

## Initial layout

- Public UI: Next.js App Router, TypeScript strict mode, mobile-first responsive CSS.
- API (later batch): Node.js/TypeScript + Fastify, modular monolith.
- Data: separate local PostgreSQL database on port 5438 and a new migration history. The baseline enables `pgcrypto` only; catalog tables will be introduced in the catalog batch. Store VND prices as integers.
- UI prototype uses clearly labeled DEMO product data. Its source imagery is a design reference and must be replaced by separately managed catalog/banner assets before production.

## Owner gates

The current screen does not create an order or claim active store policies. Do not implement checkout until the owner approves guest/account rules, payment providers, shipping prices/coverage, order lifecycle, returns/refunds, and promotion terms.

## Batch status

- Batch 0 workspace folders, approved design asset copies, local PostgreSQL Compose config, and a clean baseline migration: initialized.
- Batch 1 public homepage: visual prototype implemented; awaiting owner visual review.
- API source, catalog, cart, admin, production assets, and checkout: not started.
