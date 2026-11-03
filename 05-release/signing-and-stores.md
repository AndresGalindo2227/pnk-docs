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

---

## Applied in PANENKA Booking v0.1.0

### Android signing

| Item | Value |
|------|-------|
| Application id | `co.panenka.booking` |
| Version | `versionName 0.1.0`, `versionCode 1` |
| Keystore | `panenka-release.jks`, **outside every repository**, backed up offline |
| Alias / algorithm | `panenka`, RSA 2048, signature SHA384withRSA |
| Validity | 2026-11-02 → 2054-03-20 (beyond the 2033 minimum Google Play asks for) |
| Certificate subject | `CN=Andres Felipe Galindo Davila, OU=PANENKA Booking, O=CODEFOX SAS, L=Neiva, ST=Huila, C=CO` |
| Certificate SHA-256 | `8F:E3:67:40:42:FB:E8:76:16:96:CC:1F:15:7C:11:97:CE:D5:51:AD:54:DB:AD:96:0E:31:B8:02:03:2E:26:CB` |
| Signature schemes (APK) | v2 (v1 not needed because `minSdk` is 24) |

- `android/app/build.gradle` reads `android/keystore.properties` (git-ignored) only when it exists,
  so a clone without the key still builds debug versions but never a release one.
- Verification commands used as evidence:
  - AAB: `keytool -J-Duser.language=en -printcert -jarfile app-release.aab`
  - APK: `apksigner verify --verbose --print-certs app-release.apk`
  Both show the **same SHA-256**, so both artifacts come from the same key.
- Deviation from the rule above: CI does not sign yet. The key stays on the release owner's PC
  until it is stored as an encrypted GitHub Actions secret (planned for v0.2.0).
- Debug and release builds are signed with different keys, so Android refuses to install one over
  the other: switching between them requires uninstalling (expected).

### Google Play — simulated publication

There is no Play Console account for this course (USD 25 one-time fee). The publication is
prepared as it would be done:

1. **Upload:** Play Console → *Create app* → `PANENKA Booking`, free, category *Sports*.
2. **Play App Signing:** upload `PANENKA-Booking-v0.1.0.aab`; Google keeps the app signing key and
   our key becomes the **upload key** (lost upload keys can be reset, lost app keys cannot).
3. **Store listing:** short description, screenshots from the device test report, icon 512×512,
   feature graphic 1024×500, privacy policy URL.
4. **Data safety form:** collects name, email, phone (account) and a photo of the payment receipt
   (purchase verification); data encrypted in transit; an account-deletion request channel must exist before production.
5. **Testing track:** *Internal testing* for the PANENKA staff, then *Closed testing* with players.
6. **Production:** staged rollout `20% → 50% → 100%` watching crashes and API errors.

### iOS — simulated

Capacitor can add the iOS platform (`npx cap add ios`), but building an IPA needs macOS with Xcode
and an Apple Developer account (USD 99/year). The steps would be: signing with an Apple
Distribution certificate and an App Store provisioning profile → *Product › Archive* → upload to
App Store Connect → TestFlight → review. The web channel (`panenka-booking.web.app`) covers iPhone
users in the meantime.
