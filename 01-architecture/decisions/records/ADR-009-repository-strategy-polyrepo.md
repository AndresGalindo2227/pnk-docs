# ADR-009 — Repository strategy: one repository per deployable unit (polyrepo)

- **ID:** ADR-009
- **Date:** 2026-11-04 (records a decision applied since 2026-10-26)
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

The architecture chosen for the course is **distributed**: a mobile/web client, a REST API, a database
shared with InvFox and the documentation that governs them (ADR-005, ADR-006). Each part is built with a
different stack, is released on a different channel and changes at a different pace. The course asked to
split the project into several repositories; this ADR records **why** the split follows the deployable
units and what it costs.

**Known constraints:**
- Single developer; the professor reviews every repository against `pnk-docs`.
- Different toolchains: Node/Angular/Gradle (app), PHP (API), SQL (database), Markdown/OpenAPI (docs).
- The API is integrated with the existing InvFox database, which must not be modified by the app team.

---

## Decision

**We decided:** use **one repository per deployable unit**, all public under `AndresGalindo2227`:

| Repository | Deploys to | Versioned as |
|------------|-----------|--------------|
| `pnk-app` | Android (signed APK/AAB) and Firebase Hosting (PWA) | `v0.1.0` + GitHub Release with binaries |
| `pnk-api` | PHP server next to the InvFox database | `v0.1.0` |
| `pnk-db` | Migrations applied to the InvFox MySQL database | `v0.2.0` |
| `pnk-docs` | Governance, ADRs and the OpenAPI contract (the source of truth) | `v0.1.0` |
| `pnk-portal` (planned) | Admin web portal (ADR-006) | — |

The repositories are tied together by **contracts, not by shared code**: `openapi.yaml` in `pnk-docs`
defines the API that `pnk-app` consumes and `pnk-api` implements, and the migrations of `pnk-db` define the
tables the API uses. Every repository follows the same `develop / qa / main` flow (`git-conventions.md`).

**Justification:** each unit gets its own history, CI, permissions and release cadence. A change in the
database or the API is reviewed in a PR of its own, and a release of the app never drags unrelated
backend changes. It also mirrors how PANENKA will operate: the API and migrations live on the server
next to InvFox, while the app ships through stores and hosting.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| Polyrepo by deployable unit (chosen) | Independent releases, CI and history; small focused PRs | Cross-repo changes need several PRs; contract drift must be watched | — (chosen) |
| Monorepo (Nx/pnpm workspaces) | One PR for cross-cutting changes, shared tooling | Mixes PHP, SQL and Node toolchains; one CI for everything; does not match the deployment units | Tooling cost higher than the benefit for one developer |
| App and API in one repository, docs apart | Fewer repositories | Releases of app and API coupled; harder to give access per component | Breaks the distributed architecture of the course |

---

## Consequences

**Positive:**
- Each release is traceable to one repository and one tag (`pnk-app v0.1.0`, `pnk-db v0.2.0`).
- CI can be tailored per stack (today `pnk-app` has GitHub Actions; `pnk-api` runs `tests/run.php` locally).

**Negative / Trade-offs:**
- A feature that touches app + API + DB needs up to three PRs (e.g. idempotency keys: pnk-db #4,
  pnk-api #2, pnk-app #6).
- The contract can drift from the code; mitigated by API First (OpenAPI before code) and by reviewing
  `pnk-docs` in the same sprint.

**Impact on the system:**
- Documents to update: `00-governance/git-conventions.md` (repository table), this register.
