# ADR-004 — Booking consistency and offline strategy

- **ID:** ADR-004
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

PANENKA has one soccer-9 field (`Cancha 9`) that splits into three soccer-5 fields (`Cancha 5A`, `5B`, `5C`)
built on three physical sectors (A, B and C, table `field_units`). Booking `5A` at 18:00 must block `9` at 18:00, and booking `9` must block
all three soccer-5 fields. Two players can try to book the same slot at the same second, and players
often have weak mobile signal at the venue.

**Known constraints:**
- The source of truth is the InvFox MySQL database (InnoDB transactions available).
- A double booking is the worst possible failure for the business.
- Writes made offline cannot be validated against other players' bookings.

---

## Decision

**We decided:**

1. **Consistency in the database, not in the app.** Each booking inserts one row per occupied
   **sector** into `booking_slots` with primary key `(unit_id, slot_start)`, inside one transaction.
   The map `field_unit_map` says which sectors each field occupies: `Cancha 9` → A+B+C; `Cancha 5A` → A only. A duplicate key (SQLSTATE 23000) rolls back the transaction
   and the API answers **409 `SLOT_ALREADY_TAKEN`**.
2. **Writes are online-only.** Creating or cancelling a booking requires connectivity; the app shows the
   offline state and disables the button instead of queuing the request.
3. **Reads are cached.** "My bookings" is stored locally (Capacitor Preferences) after each successful
   load and shown with a "last updated" label when offline.
4. **Payment receipt photos are queued.** A receipt captured offline is stored locally and uploaded when
   the connection returns, because uploading it later does not create conflicts.

**Justification:**
A primary key is checked atomically by the database engine, so no race condition in PHP or in the app can
create a double booking. Online-only writes avoid promising a slot the player may not get.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| Sector rows with PK + online-only writes (chosen) | Atomic, simple, honest UX | No offline booking | — (chosen) |
| Check availability with `SELECT` then `INSERT` | Easy to write | Race condition between the two statements | Allows double booking |
| `SELECT ... FOR UPDATE` locks | Correct | More complex, deadlock risk across sectors | PK is simpler and enough |
| Offline queue for bookings with later sync | Works without signal | Conflicts discovered after the player leaves | Unacceptable UX for paid slots |

---

## Consequences

**Positive:**
- Double booking is impossible by design, verified with a concurrency test.
- The split/merged field rule lives in one table (`field_unit_map`).

**Negative / Trade-offs:**
- No booking without connectivity.

**Impact on the system:**
- Affected features: `bookings`, `my-bookings`, `payments`.
- Documents to update: [`../../architecture-overview.md`](../../architecture-overview.md) and the OpenAPI contract in `03-api-and-data`.
