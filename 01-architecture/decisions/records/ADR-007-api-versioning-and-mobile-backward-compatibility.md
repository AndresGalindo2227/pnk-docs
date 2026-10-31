# ADR-007 — API versioning and mobile backward compatibility

- **ID:** ADR-007
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

A web deploy updates every user at once, but mobile users update the app whenever they want. An old app
version may call the API months after a release, so a breaking change in the API could break bookings for
players who did not update.

---

## Decision

**We decided:**
1. Every endpoint lives under **`/api/v1`**. A breaking change creates `/api/v2`; `v1` stays alive at least
   until the minimum supported app version uses `v2`.
2. Inside a version, changes are **additive only** (new optional fields, new endpoints). Clients must
   ignore unknown fields.
3. The app sends its version in the `X-App-Version` header. `GET /api/v1/config` returns
   `minAppVersion`; if the installed app is older, it shows a blocking "please update" screen.
4. Releases follow **SemVer** and the contract version in `openapi.yaml` follows the API version.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| URL versioning + minimum app version (chosen) | Visible, easy to route in PHP | URLs change on major versions | — (chosen) |
| Header-based versioning (`Accept: ...v2`) | Clean URLs | Harder to test and to route in shared hosting | Less practical |
| No versioning | Zero effort | Any change can break old apps | Unacceptable for mobile |

---

## Consequences

**Positive:** old apps keep working; forced update is possible for security fixes.

**Negative / Trade-offs:** two API versions may need maintenance at the same time.
