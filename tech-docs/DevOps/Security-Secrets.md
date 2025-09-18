# Security & Secrets — Guidance

Some commits show example secrets in ecosystem.config.js for local/testing. In production:

- Do not commit real secrets. Load them from environment variables or a secret manager (e.g., Vault, AWS SSM, Kubernetes secrets).
- Keep Vuexy and Flowise HS256/JWT shared secrets in sync across services where signature validation is required.
- Rotate secrets regularly and after any suspected exposure.
- For local dev, prefer a `.env.local` or pm2 ecosystem file that is gitignored; never reuse production secrets locally.

Relevant settings:
- OAUTH_STATE_SECRET, SHARED_SSO_SECRET / JWT_SECRET, JWT_AUTH_TOKEN_SECRET
- APP_URL, NEXTAUTH_URL, FLOWISE URLs
- Gupshup partner/app/account keys (Flowise)

If in doubt, replace hard-coded values with `process.env.*` and document the required envs in README or the service-specific docs.
