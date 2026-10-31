# ADR-002 — Navigation: central route table with functional guards

- **ID:** ADR-002
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

Most screens require a logged-in player and some are only for guests (login, sign-up). If each page
checks the session by itself, the check is easy to forget and protected content can flash before the
redirect.

**Known constraints:**
- Angular 22 standalone components (no NgModules).
- The route list must match `navigation.md`, which reviewers use to verify the app.

---

## Decision

**We decided:** keep **one route table** in `app.routes.ts`, lazy-load pages with `loadComponent`, and
protect groups of routes with **functional guards** (`authGuard`, `guestGuard`) declared in the table
with `canMatch`.

**Justification:**
Access rules are visible in one place, a protected page is never even loaded for an anonymous user, and
the table maps one-to-one to the documented route registry.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| Central table + functional guards with `canMatch` (chosen) | Single source of truth, page not loaded when denied | — | — (chosen) |
| Session check inside each page `ngOnInit` | No guard code | Easy to forget, content flashes, duplicated logic | Unsafe and inconsistent |
| Class-based guards | Familiar from older Angular | Deprecated style in modern Angular | Not idiomatic in Angular 22 |
| Nested `loadChildren` route files per feature | Scales for large apps | Spreads routes over many files | Unnecessary for 7 routes |

---

## Consequences

**Positive:**
- One file to review for access control.
- Route registry in `navigation.md` stays verifiable.

**Negative / Trade-offs:**
- The table grows with every new screen and must be kept ordered (static segments first).

**Impact on the system:**
- Documents to update: [`../../navigation.md`](../../navigation.md).
