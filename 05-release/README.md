# 05 — Release

> **What is this?** How the app is built, signed, distributed to the stores, and observed in
> production.

## Why this section exists

Release is where a mobile app differs most from a backend: store review, signing identities,
staged rollouts, and forced updates. These rules make releases repeatable and reversible.

---

## Documents in this section

| File | Purpose |
|------|---------|
| [ci-cd.md](./ci-cd.md) | Pipeline stages from commit to store |
| [deployment.md](./deployment.md) | Cloud architecture, environments and runbooks (v0.1.0) |
| [signing-and-stores.md](./signing-and-stores.md) | Signing identities and store distribution |
| [observability.md](./observability.md) | Crash reporting, analytics, feature flags |
| [_template-release-checklist.md](./_template-release-checklist.md) | Per-release go/no-go checklist |
| [releases/v0.1.0.md](./releases/v0.1.0.md) | Go/no-go record of v0.1.0 |

---

## Correlations with other sections

| This section depends on... | Why |
|----------------------------|-----|
| `00-governance/git-conventions.md` | Versioning and tags trigger the release |
| `04-quality/` | Tests and performance budgets gate the release |
| `03-api-and-data/` | Breaking API changes may require a forced update |

---

## Questions this section must answer

- How does a commit become a store build?
- Where do signing credentials live?
- How is a rollout staged and rolled back?
- How is production health observed?
