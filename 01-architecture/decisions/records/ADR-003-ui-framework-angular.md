# ADR-003 — UI framework: Angular

- **ID:** ADR-003
- **Date:** 2026-10-18
- **Status:** Accepted (2026-10-31)
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

Ionic supports Angular, React and Vue. The app needs layers with ports and adapters, an HTTP client with
interceptors (token, refresh, timeout, errors) and has a single developer, so the framework should bring
those pieces out of the box.

---

## Decision

**We decided:** use **Angular 22** (standalone components, signals, `inject()`) with Ionic 9 and Capacitor 8.

**Justification:** weighted decision matrix (score 1–5 × weight; maximum 60):

| Criterion | Weight | Angular | React | Vue |
|-----------|--------|---------|-------|-----|
| Ionic and Capacitor integration | 3 | 5 (15) | 4 (12) | 4 (12) |
| Built-in structure and dependency injection for ports | 3 | 5 (15) | 2 (6) | 3 (9) |
| First-class strict TypeScript | 2 | 5 (10) | 4 (8) | 4 (8) |
| Learning curve for the author (PHP OOP background) | 2 | 3 (6) | 3 (6) | 4 (8) |
| Included tools (CLI, router, HttpClient, interceptors, testing) | 1 | 5 (5) | 3 (3) | 4 (4) |
| Ecosystem and job market | 1 | 4 (4) | 5 (5) | 4 (4) |
| **Total** | | **55** | **40** | **45** |

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| Angular (chosen) | DI, router, HttpClient and interceptors included | More boilerplate | — (chosen) |
| React | Largest ecosystem | DI and structure depend on external libraries chosen and maintained by us | Weaker fit for ports/adapters |
| Vue | Gentle learning curve | No built-in interceptors or DI | Second place in the matrix |

---

## Consequences

**Positive:** layer rules are easier to enforce with Angular DI; one framework for app and portal.

**Negative / Trade-offs:** more initial ceremony than Vue.

**Follow-up:** smooth scrolling and start-up time are measured on a mid-range Android device in
Sprint 2 against the budgets of `04-quality/performance-budgets.md`. If they are not met, a new ADR
supersedes this one; this record is not edited.

**Libraries chosen with the framework:**

| Concern | Choice | Discarded |
|---------|--------|-----------|
| UI state | Angular Signals in one store per feature (ADR-001) | NgRx Store, NgRx SignalStore |
| Async / HTTP | RxJS through `HttpClient` and interceptors | `fetch` wrappers, Axios |
| Navigation | Angular Router, one route table, functional guards (ADR-002) | Per-page session checks |
| Native | Capacitor 8 plugins behind ports | Cordova plugins |
