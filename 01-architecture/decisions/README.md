# ADRs — Architecture Decision Records

ADRs document important architectural decisions. Each file = one decision.

## How to create an ADR

1. Copy `_template-adr.md`.
2. Name it `records/ADR-NNN-short-title.md` (e.g. `ADR-002-state-management.md`).
3. Fill it in completely — especially the evaluated alternatives.
4. Once accepted, the status is **permanent** (it is not deleted; it is "Superseded" by another ADR).

## Possible statuses

- `Proposed` — under discussion
- `Accepted` — approved by the team
- `Rejected` — evaluated and discarded (document why)
- `Superseded` — replaced by ADR-NNN (indicate which)

## ADR register

| # | Title | Status | Date |
|---|-------|--------|------|
| [ADR-001](./records/ADR-001-state-management-pattern.md) | State management pattern (MVVM store + signals) | Accepted | 2026-10-18 |
| [ADR-002](./records/ADR-002-navigation-route-table-and-guards.md) | Navigation: central route table with functional guards | Accepted | 2026-10-18 |
| [ADR-003](./records/ADR-003-ui-framework-angular.md) | UI framework: Angular | Accepted | 2026-10-31 |
| [ADR-004](./records/ADR-004-booking-consistency-and-offline-strategy.md) | Booking consistency and offline strategy | Accepted | 2026-10-18 |
| [ADR-005](./records/ADR-005-backend-rest-api-integrated-with-invfox.md) | Backend: REST API integrated with InvFox | Accepted | 2026-10-18 |
| [ADR-006](./records/ADR-006-web-channels-pwa-and-admin-portal.md) | Web channels: PWA for players and Angular admin portal | Accepted | 2026-10-18 |
| [ADR-007](./records/ADR-007-api-versioning-and-mobile-backward-compatibility.md) | API versioning and mobile backward compatibility | Accepted | 2026-10-18 |
| [ADR-008](./records/ADR-008-on-device-storage.md) | On-device storage: secure storage, Preferences and no local database | Accepted | 2026-10-31 |

> Add a row here as you create ADRs in `records/`.
