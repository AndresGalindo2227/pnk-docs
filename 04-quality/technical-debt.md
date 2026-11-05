# Technical Debt Register — v0.1.0

> Known shortcuts and gaps, why they exist and when they will be paid. Reviewed at every release.

| ID | Debt | Why it exists | Risk | Plan |
|----|------|---------------|------|------|
| TD-01 | API runs on the PANENKA PC behind ngrok | No server or domain during the course (ADR-010) | API down when the PC is off | v0.2.0: VPS + `api.panenka.co` |
| TD-02 | No certificate pinning | The TLS certificate belongs to ngrok | Man-in-the-middle with a trusted rogue CA | After TD-01 |
| TD-03 | Release not minified (R8 off) | Plugins' keep rules not reviewed yet | Easier reverse engineering, larger APK | v0.2.0 |
| TD-04 | The app ignores `GET /config` `minAppVersion` | API side done; the blocking "update" screen was not built (ADR-007) | Old apps keep calling after a breaking change | v0.2.0 |
| TD-05 | Signing outside CI | Keystore not stored as an encrypted secret yet | Manual, error-prone releases | v0.2.0: `release.yml` on tags |
| TD-06 | `pnk-api` has no CI; 10 tests run by hand | Time; CI started with the app | Regressions in business rules | v0.2.0: GitHub Actions for PHP |
| TD-07 | No automated concurrency test against MySQL | Verified manually on the device | Sector rule regressions under load | v0.2.0 |
| TD-08 | Playwright E2E configured but not in the gate | Needs the API running in CI | UI flows only tested by hand | After TD-06 |
| TD-09 | Admin portal and Wompi not built (HU-11, HU-10) | Capacity went to the *Must* stories | Receipts reviewed in the database | v0.2.0 |
| TD-10 | No crash reporting | Not in the v0.1.0 scope | Failures in the field go unnoticed | v0.2.0: Crashlytics |
| TD-11 | Past slots shown as *Ocupado* | Same style as taken slots | Confusing for players | Show "—" for past hours |
| TD-12 | `bookings/presentation/booking-detail.page.ts` imports the `ReceiptQueue` / `ReceiptSync` ports from `payments/domain` | The detail needed to show "receipt queued" quickly | Couples two features; the ESLint rule does not cover presentation | Move both ports to `core/` and add a cross-feature lint rule |
