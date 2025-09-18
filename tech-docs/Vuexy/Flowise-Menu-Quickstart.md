# Flowise Menu Quickstart (for junior devs)

Goal: make the "AI Chatbot" menu open Flowise with the logged-in user.

- Link in UI: `src/components/layout/vertical/VerticalMenu.tsx` uses href `/flowise`.
- Rewrites: `next.config.ts` rewrites `/flowise` and `/:lang/flowise` to `/api/flowise/sso`.
- Server route: `src/app/api/flowise/sso/route.ts` reads the session, builds a signed URL, and redirects to Flowise.
- Fallbacks: If user has no email (common for Facebook), the system synthesizes one so Flowise can identify the user.

How to test:
1) Login in Vuexy (email or Facebook).
2) Click the brain icon in the left menu.
3) You should land on Flowise Chatflows as your user. If not, check server logs for `flowise-sso` trace and verify secrets/URLs.

Where to configure:
- Flowise URL envs: `NEXT_PUBLIC_FLOWISE_URL` (preferred) or fallback in the SSO route.
- Secrets: Must match Flowise server config for signature verification.
