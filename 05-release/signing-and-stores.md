# Signing & Store Distribution

## Signing

- Store signing credentials (keystore / provisioning profiles / certificates) in **CI secret
  storage**, never in the repository.
- Android: one release keystore, access restricted to release owners.
- iOS: managed signing with a dedicated distribution certificate.
- Rotate credentials on team changes; document who holds access.

## Versioning

- Bump SemVer version + build number before every store upload
  (see [`00-governance/git-conventions.md`](../00-governance/git-conventions.md)).
- The build number is monotonic and never reused.

## Store rollout

- Ship via **staged rollout**: e.g. `5% → 20% → 50% → 100%`.
- **Halt** the rollout on a crash/ANR spike or a critical bug report. [CASE-002]
- Keep store metadata (screenshots, changelog, privacy labels) versioned with the release.

## Forced update

- Maintain a remote **minimum-supported-version** flag.
- When a release contains a breaking API change or critical fix, raise the minimum version to
  force older clients to update.

## Rollback

- The previous build stays available.
- Define the rollback trigger (crash-free below threshold, payment breakage, etc.) and who
  makes the call.

---

**Evidence:** [CASE-002](https://github.com/jesusarielgb-works/engineering-casebook/blob/v1.0.0/records/cases/CASE-002-crowdstrike-channel-file-291-2024.md) — an update shipped to every machine at once.
