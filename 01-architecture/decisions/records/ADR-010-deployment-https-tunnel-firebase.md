# ADR-010 — Deployment of v0.1.0: API over an HTTPS tunnel, PWA on Firebase Hosting, local signing

- **ID:** ADR-010
- **Date:** 2026-11-04 (records the decision applied on 2026-11-03)
- **Status:** Accepted (temporary; to be superseded when the API moves to its own server)
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

Until H3 the app called the API over plain HTTP on the local network (`http://192.168.1.24:8000`).
That only worked on the same Wi-Fi, broke the rule "HTTPS only in production" (`security-rules.md`) and
browsers block an HTTPS page calling an HTTP API (mixed content). The API must stay next to the InvFox
MySQL database (ADR-005), and there is no budget for a domain or a server during the course.

**Known constraints:**
- Free tools only; the release had to work on 4G for the demo on 2026-11-06.
- Signing keys must never be in a repository (`security-rules.md`).

---

## Decision

**We decided:**
1. **API:** publish `pnk-api` through an **ngrok static domain** (`https://hangout-frown-salt.ngrok-free.dev`).
   The PC opens an outbound tunnel; ngrok terminates TLS with a valid certificate. The app sends
   `ngrok-skip-browser-warning` and the API allows it in CORS.
2. **Web:** publish the PWA on **Firebase Hosting** (`https://panenka-booking.web.app`), with versioned
   releases and one-click rollback.
3. **Android:** sign release builds **locally** with the PANENKA key (outside every repository) and attach
   the signed APK/AAB to the GitHub Release of each tag.
4. **Cleartext traffic is disabled** in release builds; it is enabled only for the LAN demo build with
   `PNK_LAN=1`.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| ngrok static domain + Firebase Hosting (chosen) | Free, fixed HTTPS URL, no router changes, API stays next to InvFox | Depends on the PC being on; free plan limits | — (chosen, temporary) |
| Free PHP hosting with MySQL | Always on | Slow and unstable; database separated from InvFox | Breaks ADR-005 |
| VPS with own domain | Production-grade, allows certificate pinning | Cost and setup time | Planned for v0.2.0 |
| Cloudflare named tunnel | Stable, free | Requires an own domain on Cloudflare | No domain available |

---

## Consequences

**Positive:**
- The app works on any network over HTTPS; the same build serves Android and the web.
- Only `environment.prod.ts` and the CORS list change when the API moves to its own server.

**Negative / Trade-offs:**
- The API is available only while the PANENKA PC, MySQL and the tunnel run (runbook in
  `05-release/deployment.md`).
- Certificate pinning is not possible while the certificate belongs to ngrok.
- Signing is not done in CI yet (deviation from `05-release/ci-cd.md`).

**Impact on the system:**
- Documents: `05-release/deployment.md`, `05-release/signing-and-stores.md`, `04-quality/mobile-security.md`.
