# Mobile Security (Runtime Checklist)

Complements [`00-governance/security-rules.md`](../00-governance/security-rules.md) with
checks run before each release.

## Storage

- [ ] Tokens and secrets only in secure storage (Keychain / Keystore).
- [ ] Sensitive local data encrypted; cleared on logout.
- [ ] No PII cached that the app does not need.

## Transport

- [ ] All traffic over TLS; no cleartext in release builds.
- [ ] Certificate pinning enabled for sensitive APIs (auth, payments).

## Permissions

- [ ] Only minimum runtime permissions requested, just-in-time, with rationale.
- [ ] No unused permissions declared in the manifest/plist.

## Build

- [ ] Release build obfuscated/minified; debug logs stripped.
- [ ] Debug flags and verbose logging disabled in production.
- [ ] No API keys or credentials in the bundle or repository.

## Dependencies

- [ ] Dependency vulnerability scan run; known-vulnerable packages patched. [CASE-004]
- [ ] Third-party SDKs reviewed for what data they collect.

---

**Evidence:** [CASE-004](https://github.com/jesusarielgb-works/engineering-casebook/blob/v1.0.0/records/cases/CASE-004-equifax-2017.md) — one unpatched component exposed 145.5 million records.

---

## Status in PANENKA Booking v0.1.0

| Check | Status | How / why |
|-------|--------|-----------|
| Tokens only in secure storage | ✅ | `@aparajita/capacitor-secure-storage` (Android Keystore). On the web they live **only in memory**, so the session ends on reload (ADR-008) |
| Local data cleared on logout | ✅ | `LocalCache.clearAll()` wipes every `pnk.cache.*` key (bookings, booking detail, receipt queue) |
| No PII cached that is not needed | ✅ | Only the player's own bookings and pending receipts are cached |
| TLS only, no cleartext in release | ✅ | The release build calls `https://…/api/v1`; `cleartext` is **off** unless the LAN demo is built with `PNK_LAN=1` |
| Certificate pinning | ❌ Planned | Not applied while the API certificate belongs to ngrok; planned when the API has its own domain |
| Minimum permissions, just-in-time | ✅ | Camera asked when tapping *Tomar foto*, notifications on the first booking; gallery works without the camera permission. Each permission in `AndroidManifest.xml` has a comment with its reason |
| No unused permissions | ✅ | `INTERNET`, `ACCESS_NETWORK_STATE`, `CAMERA`, `READ_MEDIA_IMAGES`, `READ_EXTERNAL_STORAGE` (≤ API 32), `POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM` — all used |
| Backups disabled | ✅ | `android:allowBackup="false"` so cached data is not copied to cloud backups |
| Release minified / obfuscated | ❌ Planned | `minifyEnabled false` in v0.1.0; R8 is planned for v0.2.0 after checking the plugins' keep rules |
| No secrets in the bundle or repository | ✅ | `config.php` (DB password, JWT secret), `keystore.properties` and `*.jks` are git-ignored and verified with `git check-ignore` |
| Dependency scan | ✅ | `npm audit` reviewed; `npm audit fix --force` is never run because it can break the framework versions |
| Server-side rules | ✅ | The API, not the screen, enforces: sector rule, slot in the past, 3-hour cancellation window, one receipt in review, receipt type checked by its bytes (`finfo`), other players' bookings answer `404` |
| CORS allow-list | ✅ | Only the app origins (`http://localhost`, `capacitor://localhost`), the dev server (`http://localhost:8100`) and the Firebase Hosting origins |
| Security headers on the web | ✅ | Firebase Hosting sends `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy` |
