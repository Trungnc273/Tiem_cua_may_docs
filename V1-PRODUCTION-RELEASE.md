# Midora V1 production release

Status: **accepted; production deployed; technical shop-open gate passed**  
Verified: 2026-10-04

## Production identity

- Brand: Midora
- Storefront: <https://midora.pro.vn>
- `www` host: redirects to the root host
- Production is running the verified release source commits below. Main-branch merge commits do not change the deployed source.

## Verified release source

| Component | Release commit |
| --- | --- |
| Storefront (FE) | `e37022336edb4a657881af2710b9f5f24f60d540` |
| API (BE) | `96d58a9b4f52c106177733eef58e82308bd8cc93` |

## Owner-approved production gates

- Functional V1: PASS
- PostgreSQL integration: 17/17 PASS
- Playwright: 390×844 and 1440×900 PASS
- Cloudflare R2 media: PASS
- Brevo order notification: PASS
- Root host, `www` redirect, order flow, and stock restoration: PASS
- Production health: healthy
- SUMFLOW: unchanged and healthy

Shipping uses an estimated range where configured; staff confirms the final fee. Product catalog content remains managed through Admin. An empty “Mới về” section is expected until an actual product is marked “Mẫu mới”; do not add fake products to fill it.

No credentials, tokens, or database connection details belong in this document.
