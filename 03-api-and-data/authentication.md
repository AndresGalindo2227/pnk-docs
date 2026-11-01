# Authentication

Server side: JWT access token (15 min) + opaque refresh token (7 days), stored hashed and rotated
on every use (ADR-005). This document covers the client.

## Token storage

- Access and refresh tokens are stored **only** in platform secure storage through the
  `TokenStore` port, implemented with `@aparajita/capacitor-secure-storage`:
  **Android Keystore** on Android and **iOS Keychain** on iPhone (ADR-008).
- Never in `localStorage`, `sessionStorage`, Capacitor Preferences, files or IndexedDB.
- **PWA:** browsers have no Keystore, so on the web the refresh token is not persisted — the
  player logs in again when the tab is closed. Documented trade-off.
- The access token is also kept **in memory** while the app runs, to avoid reading secure storage on
  every request.

## Attaching tokens

- The access token is attached by `authInterceptor` only — repositories and pages never touch it.
- Public endpoints (`/health`, `/config`, `/fields`, `/availability`, `/auth/*`) are sent without
  the header.

## Refresh

1. A request returns `401 UNAUTHORIZED`.
2. `authInterceptor` calls `POST /auth/refresh` with the stored refresh token.
3. On success it saves the **new pair** (rotation) and retries the original request **once**.
4. Concurrent `401`s share one refresh: the interceptor keeps a single in-flight refresh
   observable (`shareReplay(1)`), so three parallel requests trigger one refresh, not three.
5. If the refresh answers `401 SESSION_EXPIRED`, or the retried request fails with `401` again, the
   session is closed (see sign-out) and the user goes to `/login` with the message
   "Tu sesión expiró. Inicia sesión de nuevo."

## Sign-out

- Call `POST /auth/logout` (best effort), then delete both tokens from secure storage, clear the
  in-memory token, the cached *Mis reservas* and any queued receipt uploads.
- Cancel in-flight authenticated requests and navigate to `/login` with `replaceUrl: true`.

## Rules

- Never log tokens or credentials.
- No long-lived secrets in the app: no API keys, no Wompi private key (the checkout is created by
  `pnk-api`).
- Passwords are only sent over HTTPS to `/auth/login` and `/auth/register`; they are never stored.
- Biometric unlock is out of scope for V1.
