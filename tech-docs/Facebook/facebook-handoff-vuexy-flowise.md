# Facebook Handoff (Vuexy -> Flowise) — Updated Behavior

Audience: new devs. Goal: explain how our Facebook login handoff behaves when user email is missing and how it integrates with Flowise SSO.

## Files
- src/app/api/auth/handoff/facebook/route.ts

## What changed
- If NextAuth session has no email (common with Facebook), we now synthesize a stable pseudo-email derived from providerAccountId, e.g. `fb_<providerAccountId>@facebook.local`. If providerAccountId is missing but userId exists, we can fall back to `fb_user_<userId>@facebook.local`.
- This ensures downstream systems (Flowise SSO, workspace provisioning) treat the user as a unique account even when Facebook doesn't provide an email.

## User flow (as the user sees it)
1) User clicks "Login with Facebook" in Vuexy.
2) Facebook authenticates the user, sometimes without email permission.
3) Our handoff route checks the session; if email is missing, we synthesize one.
4) We proceed with the rest of onboarding/SSO as if an email exists.

## Why we need this
- Many FB accounts do not expose email; Flowise and other services require a stable unique identifier per user. The pseudo-email solves this reliably with low friction.

## Expected outcomes
- No more broken SSO or provisioning when Facebook hides email.
- Users appear in reporting and storage with a stable identifier.

## Security & collisions
- providerAccountId is unique per Facebook user across our app, so collisions are practically avoided.
- The `@facebook.local` domain makes it obvious this is synthetic (not routable mail).

## Related docs
- docs/tech-docs/Flowise/SSO.md (uses the session email to identify the user on Flowise side)
