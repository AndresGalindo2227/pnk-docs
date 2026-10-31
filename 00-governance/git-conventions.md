# Git Conventions

These rules apply to every PANENKA Booking repository. All repository content, branch names,
commits and Pull Requests are written **in English**.

## Repositories and permanent branches

| Repository | Permanent branches | Why |
|------------|--------------------|-----|
| `pnk-docs` | `main` only | Single source of truth: there is one current version of the governance, never one per environment |
| `pnk-api`, `pnk-app`, `pnk-portal`, `pnk-db`, `pnk-infra` | `develop`, `qa`, `main` | One branch per environment |

| Branch | Environment | Content |
|--------|-------------|---------|
| `develop` | Development | Latest integrated work; may contain features still being validated |
| `qa` | Quality assurance | Changes selected for testing before release |
| `main` | Production | Only approved, released changes; every release is tagged |

All permanent branches are protected by a GitHub ruleset: changes only through a Pull Request,
no deletions, no force pushes.

> **Adaptation of the course framework.** The reference framework proposes trunk-based development
> on `main`. This project follows the course rule of three environment branches with cherry-pick
> promotion, because it lets each environment receive only the changes selected for it.

## Temporary branches

Work never happens on a permanent branch. Each change lives in a short-lived branch:

| Type | Use | Created from (code repos) | Created from (`pnk-docs`) |
|------|-----|---------------------------|---------------------------|
| `feat/<scope>` | New functionality | `develop` | — |
| `fix/<scope>` | Bug fix | `develop` | `main` |
| `docs/<scope>` | Documentation | `develop` | `main` |
| `refactor/<scope>`, `test/<scope>`, `chore/<scope>` | Refactoring, tests, maintenance | `develop` | `main` |
| `promote/qa-<yyyymmdd>` | Promotion to QA | `qa` | — |
| `promote/main-v<x.y.z>` | Promotion to production | `main` | — |

Examples: `feat/booking-create`, `fix/token-refresh-loop`, `docs/adr-004-offline-strategy`.

## Change flow

```
feat/booking-create --PR (squash)--> develop
                                        | cherry-pick selected commits
promote/qa-20261105  --PR----------->  qa
                                        | cherry-pick validated commits
promote/main-v1.0.0  --PR----------->  main  --> tag v1.0.0
```

1. Create the temporary branch from `develop`, commit, push and open a PR to `develop`.
2. **Squash-merge** the PR: `develop` receives one commit per story or fix.
3. To promote to QA, create `promote/qa-<date>` from `qa` and apply the selected commits with
   `git cherry-pick <sha>`; open a PR to `qa`.
4. To release, do the same from `main` with `promote/main-v<x.y.z>`, open a PR to `main`, merge it
   and tag the release.

**Why cherry-pick and not merging whole branches:** environment branches are never merged blindly.
Only the specific commits that were validated move forward, unfinished work stays behind, and every
promotion is a reviewed PR that records exactly which commits reached each environment.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/): `type(scope): summary`

- Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `build`, `ci`.
- Scopes: the feature or layer touched, e.g. `auth`, `bookings`, `availability`, `payments`,
  `db`, `governance`, `adr`.
- Imperative mood, English, at most 72 characters in the summary line.

Examples:

```
feat(bookings): reject a slot when one of its field sectors is taken
fix(auth): retry the original request after refreshing the access token
docs(adr): record ADR-004 offline strategy
```

## Pull Requests

- One PR = one story, one fix or one promotion. Keep it reviewable (under ~400 changed lines).
- The PR description links the story (`Closes AndresGalindo2227/pnk-docs#N`) and lists how it was
  tested on a device.
- Required approvals: **0**. The project has a single developer and GitHub does not allow approving
  one's own PR. The PR is still mandatory and the author completes the Definition of Done checklist
  in its description before merging. When a second developer joins, this changes to **1** approval.
- Merge method: **squash** into `develop` and `main` (`pnk-docs`); promotions keep their
  cherry-picked commits.

## Releases and versions

- [SemVer](https://semver.org/): `MAJOR.MINOR.PATCH`. The first production release is `v1.0.0`.
- Android `versionCode`: a monotonic integer, incremented on every build uploaded to a store or
  delivered as an APK/AAB.
- Every release merged into `main` is tagged `vX.Y.Z`.

## Never committed

- Secrets: `.env`, `config/config.php`, API keys, Wompi keys, JWT signing keys.
- Signing material: Android keystores (`*.jks`, `*.keystore`) and their passwords.
- Generated or downloaded code: `node_modules/`, `vendor/`, `www/`, build outputs.
- Database dumps or files with personal data of real customers.
