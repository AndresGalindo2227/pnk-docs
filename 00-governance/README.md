# 00 — Governance

> Agreements that apply to the whole PANENKA Booking project: every repository, every feature,
> every commit. Read all documents here before the first commit.

---

## Documents in this section

| File | Purpose |
|------|---------|
| [agile-conventions.md](./agile-conventions.md) | Sprints, ceremonies, estimation and the `pnk-backlog` board |
| [git-conventions.md](./git-conventions.md) | Repositories, branches (`develop` / `qa` / `main`), cherry-pick promotion, commits, PRs, versions |
| [definition-of-ready.md](./definition-of-ready.md) | Checklist for a story to enter a sprint |
| [definition-of-done.md](./definition-of-done.md) | Checklist every completed story must satisfy |
| [security-rules.md](./security-rules.md) | Non-negotiable security rules for app, portal and API |
| [_template-sprint-retro.md](./_template-sprint-retro.md) | Template to capture a sprint retrospective |

Product backlog: [pnk-backlog](https://github.com/users/AndresGalindo2227/projects/5) — user stories are
issues of this repository (`HU-01` … `HU-14`).

---

## How governance applies

Governance rules apply to **the entire project**. If a rule conflicts with a local convention,
governance wins unless an ADR (`01-architecture/decisions/records/`) overrides it.

Changing a governance rule requires a Pull Request to this repository that explains the reason.

---

## Questions this section answers

- How is work planned, estimated and closed?
- What do "ready" and "done" mean for a story?
- How are repositories, branches, commits, PRs and releases named and promoted?
- What are the non-negotiable security rules?
