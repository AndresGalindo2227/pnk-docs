# Agile Conventions

PANENKA Booking is built by a **single developer** in short iterations, using Kanban on the
[`pnk-backlog`](https://github.com/users/AndresGalindo2227/projects/5) board plus weekly sprints.
The ceremonies are kept, scaled down to a one-person team, because they leave written evidence of
every decision.

## Team and roles

| Role | Person | Owns |
|------|--------|------|
| Product owner | Andrés Felipe Galindo Dávila, with input from the PANENKA field owner (customer) | Priorities (MoSCoW), acceptance |
| Tech lead | Andrés Felipe Galindo Dávila | Architecture, ADRs, DoD enforcement |
| Developer / QA | Andrés Felipe Galindo Dávila | Implementation, tests, device validation, docs |
| Reviewer (course) | Course professor | Reviews the repositories against this documentation |

## Board workflow

```
Backlog -> Ready -> In progress -> In review -> Done
```

| Column | A card is here when… | WIP limit |
|--------|----------------------|-----------|
| Backlog | It exists and is prioritized | — |
| Ready | It meets the [Definition of Ready](./definition-of-ready.md) | — |
| In progress | Someone is implementing it (a branch exists) | **2** |
| In review | Its Pull Request is open | 5 |
| Done | It meets the [Definition of Done](./definition-of-done.md) and the PR is merged | — |

Board fields: `Priority` (Must / Should / Could / Won't), `Estimate` (story points) and
`Component` (app, api, portal, db, infra, docs — the main repository affected).

## Cadence

| Ceremony | When | Timebox | Output |
|----------|------|---------|--------|
| Sprint planning | First day of each 1-week sprint | 30 min | Stories moved to Ready / In progress |
| Daily check-in | Every working day | 5 min | Board updated; blockers written as issue comments |
| Backlog refinement | When a story is about to enter a sprint | 15 min | Story meets the DoR |
| Sprint review | Last day of the sprint | 30 min | Demo on an Android device or emulator |
| Retrospective | Last day of the sprint | 15 min | `retro-sprint-NN.md` from the template, with actions |

## Sprints

| Sprint | Dates | Goal |
|--------|-------|------|
| 1 | 2026-10-26 → 2026-11-01 | Governance, architecture, API contract, database migrations, repositories |
| 2 | 2026-11-02 → 2026-11-08 | Must stories implemented end to end, signed APK, final report |

## Estimation

- Story points on a Fibonacci scale: `1, 2, 3, 5, 8`.
- A story larger than `8` must be split before it enters a sprint.
- Points estimate relative effort, not hours.
- Prioritization uses **MoSCoW**. `Won't` stories stay in the backlog with the reason they are out of V1.

## Backlog hygiene

- Every story has a user value (As / I want / So that) and verifiable acceptance criteria
  (Given / When / Then).
- Stories that touch the UI link a prototype screen.
- Stories that call the API reference the endpoints of the OpenAPI contract
  (`03-api-and-data/`).
- A Pull Request that implements a story closes it with `Closes AndresGalindo2227/pnk-docs#N`.
