# ADR-005 — Backend: REST API integrated with InvFox

- **ID:** ADR-005
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

PANENKA already runs InvFox, a PHP 8 / MySQL point-of-sale system that the author maintains. Bookings,
payments and the cash register must end up in the same database so the owner sees one daily report.
InvFox has no API: it is a server-rendered app with session login.

**Known constraints:**
- Hosting is a classic PHP + MySQL plan; no Node or Docker in production for V1.
- The mobile app needs stateless authentication (no PHP session cookies).
- InvFox must keep working for the cashier during the whole project.

---

## Decision

**We decided:** build **`pnk-api`**, a small **PHP 8 REST API** (JSON, `/api/v1`) that shares the InvFox
MySQL database, defined **API First** by the OpenAPI 3.1 contract. Authentication uses a **JWT access token
(15 minutes)** and a **refresh token (7 days)** stored **hashed** in the database and rotated on every use.
InvFox evolves with the **strangler fig** pattern: new features go through the API; old screens keep
working until they are replaced.

**Justification:**
It reuses the author's PHP knowledge and the existing hosting, keeps one source of truth, and lets the
mobile app and the portal work against a documented contract (and a mock server) before the backend exists.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| PHP REST API on the InvFox database (chosen) | Same hosting, one database, known language | API must be written from scratch | — (chosen) |
| Firebase (Auth + Firestore) | Fast start, offline SDK | Data split from InvFox; sector rule hard to enforce atomically | Two sources of truth |
| New Laravel backend | Mature framework | Learning curve and heavier deploy within two sprints | Not feasible before Nov 8 |
| Expose InvFox AJAX endpoints directly | No new code | Session cookies, no contract, mixed concerns | Insecure and not mobile-friendly |

---

## Consequences

**Positive:** one database and one daily report; the contract enables parallel work with a mock server.

**Negative / Trade-offs:** the API must implement auth, validation and error handling by hand.

**Security notes:** passwords use `password_hash`; refresh tokens are stored as SHA-256 hashes; resources
of other players answer **404** to avoid disclosing that they exist.
