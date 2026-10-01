# Batch 1 — visual homepage report

## Workspace and project boundary

- Workspace: `D:\PRJ-2026\giang\tiem_cua_may`
- FE: `TIEM_CUA_MAY_FE/`; BE: `TIEM_CUA_MAY_BE/`; docs: `TIEM_CUA_MAY_DOC/`.
- A local workspace repository and separate FE, BE, and DOC repositories on `main`; no remotes are configured. No MIDORA remote or source domain was added to this workspace.
- Reused technical guidance only: App Router, TypeScript, responsive UI, ESLint and CI patterns. No MIDORA lighting routes, entities, migrations, or products are present.

## Implementation

- FE: Next.js App Router, Next 16.3.5 + React 19, TypeScript strict mode, CSS mobile-first. Node requirement is 22.18+ for the built-in TypeScript test runner. The lockfile is pnpm v9 format and passes frozen lockfile validation.
- Homepage: owner supplied logo; cloud blue hero; search interaction; six category shortcuts; two-column mobile demo cards with favorites and color swatches; demo campaign panel; service information; desktop navigation/footer.
- Source logo preserved at `../brand-assets/logo-original.jpg`; FE copy at `../TIEM_CUA_MAY_FE/public/brand/logo.jpg`. Demo hero and four product crops are separate files under `public/demo/`; the full design screenshot is not served by the FE.
- Vietnamese UI uses system Arial/Segoe UI; text was legible in Chromium captures. Store claims are avoided: card imagery/badges are DEMO, and shipping/returns information says policies are pending.
- BE is a fashion-domain placeholder for Fastify/PostgreSQL/Drizzle. A separate PostgreSQL 17 dev service is defined at root; migration `0000_initial.sql` enables `pgcrypto` only. No application API, catalog tables, or production database have been created yet.
- Checkout, payment, real shipping/returns rules, customer accounts, catalog, and admin remain out of this batch.

## Quality evidence

- Production build: **passed** (`pnpm build`) on the exact locked Next 16.3.5.
- TypeScript: **passed** (`pnpm typecheck`, `tsc --noEmit`).
- ESLint: **passed, 0 errors / 0 warnings** (`pnpm lint`).
- Unit tests: **passed, 4/4** (`pnpm test`) for empty search, case-insensitive matching, Vietnamese diacritics, and demo-only badges.
- Browser smoke: **passed** for empty/no-match/matching search and the favorite toggle.
- Compose: **passed** `docker compose config --quiet`; the DB container itself was not started.
- Real Chromium viewport captures show no horizontal page overflow and all six categories/four demo products:
  - [Mobile 390 × 844](../qa/screenshots/homepage-mobile-390x844.png)
  - [Mobile 430 × 932](../qa/screenshots/homepage-mobile-430x932.png)
  - [Tablet 768 × 1024](../qa/screenshots/homepage-tablet-768x1024.png)
  - [Desktop 1440 × 900](../qa/screenshots/homepage-desktop-1440x900.png)

## Local commits

- FE `main`: `ab8452b` — `feat: add demo search and focused unit tests` (includes the homepage and its test suite).
- BE `main`: `54e21e1` — `chore: start fresh commerce schema baseline`.
- Workspace `main`: `cc318cd` — `chore: initialize Tiệm Của Mây workspace`, including the four Chromium screenshots.
- DOC `main`: architecture/report commit (see repository HEAD at handoff).
- Worktree status was clean after the recorded commits; no pushes were made.

## Review status

**Batch 1 is ready for the owner's visual review.** This is a local prototype. Install FE dependencies from the lockfile before resuming development; the application/API foundations and all later commerce batches remain unbuilt.
