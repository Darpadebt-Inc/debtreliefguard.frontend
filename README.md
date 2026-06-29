# debtreliefguard.frontend
the front end for debtreliefguard.com

## Deployment

Static site on **Cloudflare Pages**. **Pushing to `main` auto-deploys to production.**
A push runs `.github/workflows/trigger-deploy.yml`, which authenticates to the backend
deploy proxy with an **ephemeral GitHub OIDC token** (this public repo stores **no
secrets**) — the backend then deploys both frontends and runs a live smoke check.

- This repo holds **no Cloudflare secrets** by design; the in-repo `deploy.yml` is an
  intentional no-op.
- Manual fallback + full rationale: [`DEPLOYMENT.md`](./DEPLOYMENT.md).
- Architecture, hardening, and one-time activation: backend
  `docs/HORIZON-SYNC-DEPLOY-TRIGGER.md`.
