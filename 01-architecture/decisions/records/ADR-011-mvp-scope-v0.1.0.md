# ADR-011 — Scope of v0.1.0: a minimum viable product limited to the player's booking flow

- **ID:** ADR-011
- **Date:** 2026-11-06 (records the scope decision applied during the sprint, 2026-10-17 to 2026-11-04)
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

The backlog has 14 user stories (52 story points) prioritised with MoSCoW. The project has one
developer and a fixed delivery date (2026-11-08). PANENKA needs to know if players will book and pay a
deposit from the phone **before** it invests in online payments, an admin portal or electronic invoicing.

The real risks of the product are technical and sit in the player's flow:

- two players must never get the same sector at the same hour (Cancha 9 = sectors A + B + C);
- the field has weak mobile signal, so the app must work with intermittent connectivity;
- the session and the payment receipt carry personal and financial data.

Some stories depend on infrastructure that v0.1.0 does not have:

- **HU-10 Wompi** needs a merchant account and a webhook endpoint that is always online. With the API on
  the developer's PC behind a tunnel (ADR-010), a missed webhook would leave a paid booking unconfirmed.
- **HU-11 admin portal** is a separate deployable (ADR-006, ADR-009) with its own roles and authentication.
- **HU-14 DIAN invoice** depends on InvFox's electronic-invoicing provider.
- **HU-12 weekly booking** needs a series model and a policy for partial conflicts.

---

## Decision

**We decided:** v0.1.0 is a **minimum viable product** that delivers the whole player flow end to end and
nothing else: **HU-01 to HU-08 (all *Must*) plus HU-09** (deposit by Nequi transfer with a receipt photo).

- The deposit is paid by bank transfer and the receipt is reviewed by PANENKA (no gateway).
- The MVP is **integrated** with the InvFox database (ADR-005) and has an **on-device cache**
  (ADR-008), so it is a real product, not a prototype with mock data.
- Everything else moves to v0.2.0 or later and is recorded as technical debt (TD-09).

| Story | Priority | v0.1.0 | Technical reason |
|-------|----------|--------|------------------|
| HU-01 – HU-08 | Must | Delivered | Core flow and its risks (concurrency, offline, session) |
| HU-09 Receipt photo | Should | Delivered | Payment without a gateway; covers the offline queue and the camera plugin |
| HU-10 Wompi | Should | v0.2.0 | Needs an always-online webhook (own server, TD-01) |
| HU-11 Admin portal | Should | v0.2.0 | Separate deployable with admin roles |
| HU-12 Weekly booking | Could | Later | Needs a booking-series model |
| HU-13 Drinks, HU-14 DIAN invoice | Won't | Not planned | Outside the booking problem / external provider |

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| MVP = player flow, Must + HU-09 (chosen) | Every risky rule built and tested; deliverable on time | No online payment, no portal | — (chosen) |
| Include Wompi in v0.1.0 | Automatic payment confirmation | Webhook on a PC behind a tunnel is unreliable; merchant account needed | A lost webhook means a paid but unconfirmed booking |
| Build the admin portal first | PANENKA reviews receipts in a UI | Second app to build, secure and deploy; the player flow would be incomplete | Value for the player comes first |
| All 14 stories, thinner | Wider feature list | No time for offline, concurrency, tests and signing | Breadth without quality |

---

## Consequences

**Positive:** the hard parts (sector rule in the database, idempotency, token refresh, offline cache and
receipt queue) are built, tested (42 unit tests, 17 device cases) and released as v0.1.0. All *Must*
stories are done: 34 of 52 story points.

**Negative / Trade-offs:** PANENKA confirms deposits by hand, the admin works from the database or InvFox,
and a player cannot pay online. These are recorded in [technical debt](../../../04-quality/technical-debt.md)
(TD-01, TD-09).

**Revisit when:** the API runs on its own server with a domain (TD-01). That unblocks Wompi (HU-10)
and the admin portal (HU-11) for v0.2.0.

---

## References

- Release scope: [`05-release/releases/v0.1.0.md`](../../../05-release/releases/v0.1.0.md)
- Backlog board: GitHub Project `pnk-backlog` (issues HU-01 … HU-14 in this repository)
- Related: ADR-004, ADR-005, ADR-006, ADR-008, ADR-009, ADR-010
