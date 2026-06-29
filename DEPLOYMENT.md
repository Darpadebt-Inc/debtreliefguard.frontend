# Deployment — debtreliefguard.frontend

> **Read this before attempting any deploy.** It explains the deploy authority and
> why it is structured this way, so future deploys follow the intended (secure) path.

## TL;DR
- This repo is **PUBLIC**. It holds **no Cloudflare secrets** and **must not**.
- Production is deployed from the **PRIVATE** repo **`Darpadebt-Inc/debthelpform.backend`**,
  workflow **"Deploy Frontends to Cloudflare Pages"** (`.github/workflows/deploy-frontends.yml`).
- That backend workflow holds the only `CLOUDFLARE_API_TOKEN` / `CLOUDFLARE_ACCOUNT_ID`
  and deploys **both** public frontends (`debthelpform.frontend`, `debtreliefguard.frontend`).

## Why route deploys through the backend (security rationale — keep this)
The Cloudflare API token can edit/deploy production. The frontends are **public**;
the backend is **private**. A production-deploy credential living in a public repo is a
real risk (broad visibility, fork/PR exfiltration, accidental exposure). Centralizing it
in one private repo gives: a single rotation point, least privilege, and no secret sprawl
across public repos. This also sidesteps the (documented-broken) Pages Git integration.

**Do NOT add `CLOUDFLARE_*` secrets to this public repo** to make the in-repo
`deploy.yml` deploy. That workflow skips here **by design**; the backend is the source of truth.

## Automated deploy (proxy-mediated — now the default)
A push to this repo's `main` now triggers a production deploy **automatically** — no manual
step required. `.github/workflows/trigger-deploy.yml` mints an **ephemeral GitHub OIDC
token** (no secret is stored in this public repo) and calls the backend proxy worker
`069-deploy-trigger`, which verifies the token (signature + strict claim allowlist) and
forwards a `repository_dispatch` to the backend **"Deploy Frontends to Cloudflare Pages"**
workflow. The manual `workflow_dispatch` path below still works unchanged as a fallback.

The forwarding credential lives only as a Cloudflare Worker secret — this repo stays
sterile. Design + one-time activation (set the worker secret, deploy the worker):
backend **`docs/HORIZON-SYNC-DEPLOY-TRIGGER.md`**.

## How to deploy manually (fallback path)
1. Merge your changes to this repo's `main` (the backend workflow deploys `main`).
2. Ensure the backend repo's Actions secrets are valid:
   - `CLOUDFLARE_API_TOKEN` — Cloudflare token with **Account → Cloudflare Pages → Edit**
   - `CLOUDFLARE_ACCOUNT_ID` — `edc1ec2ddc9e28cc26bc647ade3c091d`
   - Set at: GitHub → `Darpadebt-Inc/debthelpform.backend` → Settings → Secrets and variables → Actions
3. Run **`debthelpform.backend` → Actions → "Deploy Frontends to Cloudflare Pages" → Run workflow**.
4. The hardened workflow pins the exact Pages projects (`debthelpform-frontend`,
   `debtreliefguard-frontend`), fails RED on missing/rejected creds, and runs a
   post-deploy production smoke check (so "green" means actually live).

## Pinned Cloudflare Pages projects
| Frontend repo            | Pages project           | Domain                |
|--------------------------|-------------------------|-----------------------|
| debthelpform.frontend    | `debthelpform-frontend` | debthelpform.com      |
| debtreliefguard.frontend | `debtreliefguard-frontend` | debtreliefguard.com |

## Common failure: `code 10000 Authentication error`
The token in the backend secret is invalid/expired/revoked. Generate a **new** token
(Account → Cloudflare Pages → Edit), set it as `CLOUDFLARE_API_TOKEN` in the backend
repo secrets, then re-run the backend workflow. Caches do not need clearing: HTML here
is served `no-cache` and CSS is inline, so a successful deploy is visible immediately.
