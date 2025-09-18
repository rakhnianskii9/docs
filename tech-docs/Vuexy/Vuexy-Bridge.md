# Gupshup Onboarding Bridge (Vuexy -> Flowise)

This route lets Vuexy trigger Flowise Gupshup onboarding flows securely.

## Files
- src/app/api/gupshup/route.ts (server route)
- src/libs/gupshup.ts (client helpers used by UI)

## What changed
- New server route POST /api/gupshup accepts action commands and relays to Flowise with a signed JWT-like token (HS256), short TTL.
- Actions supported:
  - start: POST to Flowise /api/v1/gupshup/onboarding/start
  - contact: POST to Flowise /api/v1/gupshup/onboarding/contact
  - embed-link: GET Flowise /api/v1/gupshup/onboarding/embed-link
- If session email is missing, we synthesize a stable `user_<uid>@local` email so Flowise can attribute the request.
- Client lib (src/libs/gupshup.ts) now normalizes various embed link response shapes to a unified `{ embedUrl }`.

## User flow
1) A logged-in Vuexy user starts Gupshup onboarding from UI.
2) Frontend calls our POST /api/gupshup with the chosen action and payload.
3) The server reads the session email (or synthesizes), signs payload with shared secret, and calls Flowise endpoint.
4) The response returns appId and/or an embed URL; UI opens the embed link so the user can connect WhatsApp.

## Request/Response contracts
- POST /api/gupshup with JSON body:
  - { action: 'start', name?: string, contact?: {...} }
  - { action: 'contact', appId: string, contact: {...} }
  - { action: 'embed-link', appId: string }
- Returns JSON:
  - start: { appId: string, embedUrl?: string }
  - contact: { ok: true }
  - embed-link: { embedUrl: string }

## Configuration
- Flowise URL discovery mirrors Flowise SSO route (API_URL, NEXT_PUBLIC_FLOWISE_URL, etc.).
- Shared secret must match Flowise server (see ecosystem.config.js).
- Gupshup partner/app/account credentials live in ecosystem.config.js (Flowise side) and are not used directly by Vuexy.

## Edge cases
- No session: 401.
- Flowise down: we bubble 502 with message; check logs.
- Variants of embed link field names are handled in src/libs/gupshup.ts.

## Related docs
- docs/tech-docs/Flowise/SSO.md
