# ADR-001 — State management pattern

- **ID:** ADR-001
- **Date:** 2026-10-18
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

Every screen of the app (availability, booking, my bookings, payments) must show five states:
loading, loaded, empty, error and offline. Without one agreed pattern, each feature would manage
state its own way, screens would be hard to test, and reviews would discuss style instead of behavior.

**Known constraints:**
- Single developer, short timeline (two one-week sprints).
- State transitions must be unit-testable without rendering Ionic pages.
- Angular 22 offers signals natively.

---

## Decision

**We decided:** use **MVVM with one store per feature**. The store keeps an **immutable state object**
in a `BehaviorSubject` and exposes it to pages as a read-only **signal**. Pages render one state object
and delegate every action to the store, which calls the use cases.

**Justification:**
Immutable state with explicit statuses makes the five screen states visible in code and trivial to test.
Signals integrate with Angular change detection without extra libraries.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| MVVM store + immutable state exposed as signal (chosen) | Simple, testable, no extra dependency | Some boilerplate per store | — (chosen) |
| Mutable signals inside each component | Fastest to write | State spread across components, hard to test | Violates the layer rules |
| NgRx Store | Very explicit, great tooling | Heavy ceremony for three features | Overkill for the V1 scope |
| NgRx SignalStore | Modern, concise | Extra dependency to learn under a tight deadline | Not justified for V1 |

---

## Consequences

**Positive:**
- Each screen state is testable as plain TypeScript.
- One mental model across all features.

**Negative / Trade-offs:**
- A store file per feature, even for small features.

**Impact on the system:**
- Affected features/screens: all.
- Documents to update: [`../../layers-and-state.md`](../../layers-and-state.md).
