# Local Public Staging

This environment is for owner review only. It runs production-mode containers against a dedicated TEST-only PostgreSQL database and catalog. It is not a production deployment and contains no real customer records or owner-supplied product catalog.

## Local stack

`docker-compose.staging.yml` starts PostgreSQL, the Fastify API, and a standalone Next.js storefront. The database is bound only to loopback for operator migration/fixture commands. The API is private to the Compose network. The storefront is bound to `127.0.0.1:3180`. Next.js proxies `/api/*` to the API service, so browser cookies remain same-origin across the HTTPS tunnel. PostgreSQL and `/data/uploads/products` use distinct persistent named volumes. Container logs rotate at 10 MiB with three files retained.

Copy `.env.staging.example` to the ignored `.env.staging.local`; set a random stage-only database password. The example public origin starts as `https://stage.invalid` only to let local containers boot. Never use that value as a public URL or in production.

## Start and migrate

From the workspace root:

```powershell
docker compose --env-file .env.staging.local -f docker-compose.staging.yml up -d database
docker compose --env-file .env.staging.local -f docker-compose.staging.yml build api web
docker compose --env-file .env.staging.local -f docker-compose.staging.yml --profile migration run --rm migrate
```

The migration command applies only pending versioned SQL. It never drops, resets, or seeds a database. `/ready` stays unhealthy until migration `0004_product_admin_uploads.sql` is present. After seeding and admin bootstrap, start the app:

```powershell
docker compose --env-file .env.staging.local -f docker-compose.staging.yml up -d api web
```

Set `DATABASE_URL`, `CATALOG_MODE=test`, `TCM_ENVIRONMENT=staging`, and `TCM_SAFE_TEST_DATABASE=tiem_cua_may_batch5_stage` only in the local process environment when running `pnpm db:seed:test` or `pnpm admin:create`. Both commands require a local TEST/staging database name and an exact allowlist match. Fixture seed credentials remain TEST-only. Never auto-create production admins.

## HTTPS tunnel and owner review

Start the Quick Tunnel after `http://127.0.0.1:3180` is healthy:

```powershell
cloudflared tunnel --no-autoupdate --url http://127.0.0.1:3180
```

Copy its assigned HTTPS origin into both `TCM_PUBLIC_ORIGIN` and `TCM_CORS_ORIGINS` in `.env.staging.local`, then recreate the API and web services with that file. The origin allowlist accepts only that exact HTTPS origin. The API marks cart and admin cookies `Secure`, `HttpOnly`, and `SameSite=Lax`; it rejects state-changing requests without the configured `Origin`.

Playwright uses the HTTPS URL with a TEST admin account. Keep the tunnel and stage services alive for owner phone/external-network review. The temporary hostname changes when the Quick Tunnel process ends. No VPS, domain, registry credential, or Cloudflare account is assumed.

## Back up

Run `scripts/backup-staging.ps1`. It creates a timestamped PostgreSQL custom-format dump and a tar archive of the complete product-upload volume, then writes SHA-256 checksums. Treat the pair as one backup set. Monitor free disk space; each product image is capped at 8 MiB, and volume growth follows the number and size of uploaded originals.

For a future production release, run the same paired operations against the production Compose project and its production PostgreSQL/upload volume names, store both artifacts in protected off-host storage, and verify checksums. A database-only dump is incomplete.

## Restore drill

Run `scripts/restore-drill.ps1 -BackupDirectory <timestamped-backup> -Slug <uploaded-test-product-slug>` with the TEST admin email/password injected as `TCM_STAGE_ADMIN_EMAIL` and `TCM_STAGE_ADMIN_PASSWORD`. The script refuses non-staging configuration, verifies checksums, creates a new uniquely named local restore database and upload volume, restores both artifacts without overwriting existing data, starts a temporary API, and checks readiness, that TEST product and its variants, TEST admin orders/settings, and its uploaded image. It retains the isolated restore database and image volume as evidence; only its temporary API container is removed.

## Production Compose template

`docker-compose.production.yml` is a separate production runtime template. Fill `.env.production.local` from `.env.production.example`, use owner-controlled URL-safe database credentials, and pin the API and web image tags to the reviewed release SHA (or immutable registry digests). Build the FE image with `API_INTERNAL_URL=http://api:4000` so its same-origin `/api/*` rewrite targets the private API service. The DB and API have no published host ports; only the web binds to loopback for an explicitly configured Cloudflare Tunnel ingress. Run its `migration` profile as a separate step before starting `api` and `web`. It has no fixture seed service or development mounts.

Use the protected production Admin CLI flow with the production database and `CATALOG_MODE=production`; inject the password through the operator process environment. Do not use `seed-test.ts` or any TEST admin identity in production. Production database and upload-volume backups must be stored together in protected off-host storage; the local backup scripts are intentionally guarded for staging.

Before any real production launch, repeat the paired restore into a fully isolated environment using the protected production backup. Validate the same records/media, and keep the current production data untouched until an approved cutover plan exists.

## Operator gates

- Production mode requires `TCM_ENVIRONMENT=production`, `CATALOG_MODE=production`, an HTTPS `PUBLIC_APP_URL`, and an exact HTTPS `CORS_ORIGINS` allowlist.
- Staging mode requires `TCM_ENVIRONMENT=staging`, `CATALOG_MODE=test`, a local database whose name includes `test`/`stage`, and no public database port. The staging database is separately named and volume-isolated from local development.
- The store settings migration provides the approved phone and Messenger URL. The shipping amount remains unconfigured in production; checkout stays blocked until an owner/admin enters the approved amount. The staging seed's 25,000 VND value is TEST-only.
- Product photos in the staging journey are synthetic TEST fixtures. They are not presented as real shop items. Owner-supplied catalog names, prices, variant/SKU/stock details, descriptions, and photos are still needed before production catalog entry.
- The current owner review uses no production PII. Admin order detail is access-controlled; public order lookup is not provided. API request logging excludes bodies and redacts cookie/authorization headers.
- Migration runs as a separate operator command, not during API boot. PostgreSQL is internal in the public staging Compose; only the loopback-bound storefront is reachable by the local tunnel.
