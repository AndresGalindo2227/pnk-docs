# CI/CD

## Principles

- Reproducible builds **from CI only** — no store uploads from a developer laptop.
- Separate build flavors/schemes per environment: `dev`, `staging`, `prod`.
- All environment values (base URL, keys, flags) injected at build time — never hardcoded.

## Pipeline stages

1. **Lint + static analysis** — fails on warnings.
2. **Unit + widget tests** — must pass.
3. **Build signed artifacts** — `.aab`/`.apk` (Android), `.ipa` (iOS).
4. **Distribute to internal testers** — Firebase App Distribution / TestFlight.
5. **Pre-release checks** — startup time + app size captured and compared.
6. **Promote to store review** — after sign-off (see `_template-release-checklist.md`).

## Triggers

- PRs run stages 1–2.
- Merges to `main` run stages 1–4 to internal testers.
- A `vX.Y.Z` tag runs the full pipeline and creates the GitHub release
  (`.github/workflows/release.yml`).

## Environments

| Flavor | Backend | Distribution |
|--------|---------|--------------|
| dev | dev API | debug installs |
| staging | staging API | internal testers |
| prod | prod API | store |

---

## Applied in PANENKA Booking v0.1.0

Workflow: `pnk-app/.github/workflows/ci.yml` (GitHub Actions, `ubuntu-latest`, Node 24).

| Stage | Status | Detail |
|-------|--------|--------|
| 1. Lint + static analysis | ✅ in CI | ESLint (`--max-warnings 0`, layer rules), e2e typecheck, Prettier |
| 2. Unit tests | ✅ in CI | 42 Vitest tests |
| 3. Build | ✅ in CI (unsigned web build) | `ng build --configuration production`, kept 7 days as the `pnk-app-www` artifact |
| 3b. Signed Android artifacts | Manual | Built on the release owner's PC (see `signing-and-stores.md`) |
| 4. Distribution | Manual | `firebase deploy --only hosting`; APK installed with `adb` |
| 5–6. Pre-release checks / store | Simulated | Release checklist in `releases/` |

- **Triggers:** every pull request, and every push to `develop`, `qa` and `main`.
- `concurrency` cancels a run when a newer commit arrives on the same branch.
- The workflow only has `contents: read` permission (least privilege).
- Typical duration: 45 s – 1 min 30 s.
- **Next step (v0.2.0):** store the keystore as an encrypted secret and add a `release.yml`
  workflow on `vX.Y.Z` tags that builds the signed AAB and deploys to Firebase Hosting.
