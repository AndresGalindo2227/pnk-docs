# ADR-006 — Web channels: PWA for players and Angular admin portal

- **ID:** ADR-006
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

The package sold to PANENKA includes the POS, the mobile booking app and **web booking**. Some players
will not install an app, and the owner needs a screen to manage the schedule and approve payment receipts.

---

## Decision

**We decided:**
- **Players on the web** use the same `pnk-app` code built as a **PWA** (Ionic runs in the browser). No
  second player codebase.
- **Administration** lives in **`pnk-portal`**, a separate Angular app protected by `adminGuard`, which
  consumes the same `pnk-api`.

**Justification:** one codebase per audience. The player experience is identical in Android and web,
and admin screens (tables, filters) do not bloat the mobile bundle.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| PWA from pnk-app + separate admin portal (chosen) | No duplicated player code, clean separation | Two Angular projects | — (chosen) |
| Admin screens inside the mobile app | One project | Larger bundle, admin code shipped to players | Security and size |
| Booking pages inside InvFox (PHP) | Reuses existing server | Duplicates all booking logic outside the API | Breaks API First |

---

## Consequences

**Positive:** one booking flow to test; admin permissions are isolated.

**Negative / Trade-offs:** native-only features (camera, notifications) need web fallbacks in the PWA.
