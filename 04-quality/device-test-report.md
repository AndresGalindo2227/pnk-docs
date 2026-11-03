# Device Test Report — v0.1.0

> Manual test cases executed on a **real device** against the **real API** (not the mock).
> Each failed case produced a defect (DEF-NNN), a fix in a pull request and a new automated test
> where the rule could be expressed as one.

- **Build:** PANENKA Booking 0.1.0 (versionCode 1), release APK signed with the PANENKA key
- **Device / OS:** HONOR RMO-NX3 (Android)
- **Backend:** pnk-api (PHP 8, MySQL `invfox`) on the developer PC, published at
  `https://hangout-frown-salt.ngrok-free.dev` (see [`05-release/deployment.md`](../05-release/deployment.md))
- **Web channel:** `https://panenka-booking.web.app` (Firebase Hosting), same cases on the phone browser
- **Dates:** 2026-11-02 / 2026-11-03

---

## Test cases

| ID | Story | Case | Network | Expected | Result |
|----|-------|------|---------|----------|--------|
| TC-001 | HU-01 | Sign up with a phone that does not start with 3 and a 7-char password | Wi-Fi | Inline errors, no request sent | Pass |
| TC-002 | HU-01 | Sign up with valid data | Wi-Fi | Account created, enters the app | Pass |
| TC-003 | HU-02 | Log in with a wrong password | Wi-Fi | "Correo o contraseña incorrectos." | Pass |
| TC-004 | HU-03 | Open availability for today after 19:00 | 4G | Past slots not bookable | Pass |
| TC-005 | HU-03 | Book 5B at 18:00 and look at Cancha 9 at 18:00 | 4G | Cancha 9 shows *Ocupado* (sector rule) | Pass |
| TC-006 | HU-04 | Book 5A, then 5C, then Cancha 9 in a row | Wi-Fi | Each booking keeps its own field | **Fail → DEF-001 → Pass** |
| TC-007 | HU-07 | First booking: notification permission | Wi-Fi | Asked just-in-time; reminder scheduled 1 h before | Pass |
| TC-008 | HU-09 | Take the receipt photo with the camera | Wi-Fi | Camera permission asked just-in-time; preview shown | **Fail → DEF-002 → Pass** |
| TC-009 | HU-09 | Send the receipt, go back to the booking | Wi-Fi | *Pago en revisión*, no second upload offered | **Fail → DEF-003 → Pass** |
| TC-010 | HU-09 | Send the receipt in airplane mode | Offline | *Sin señal: guardamos el comprobante…* | Pass |
| TC-011 | HU-09 | Stay on that screen and turn airplane mode off | Offline → 4G | Turns into *Comprobante enviado* by itself; booking shows *Pago en revisión* | **Fail → DEF-005 → Pass** |
| TC-012 | HU-05 | Open *Mis reservas* in airplane mode | Offline | Saved copy with "actualizado hace X min" | Pass |
| TC-013 | HU-05 | Open a booking detail in airplane mode | Offline | Last saved data + offline notice, no *Cancelar* | **Fail → DEF-004 → Pass** |
| TC-014 | HU-06 | Cancel a booking more than 3 h before | Wi-Fi | Status *Cancelada*, slot free again | Pass |
| TC-015 | — | Whole flow with Wi-Fi off (4G only) through the public API | 4G | Same behavior as on Wi-Fi | Pass |
| TC-016 | — | Whole flow on `panenka-booking.web.app` in the phone browser | 4G | Same behavior; session ends on reload (by design, ADR-008) | Pass |
| TC-017 | — | Update the installed release APK with `adb install -r` | USB | Updates without losing the session (same signing key) | Pass |

---

## Defects found and fixed

| ID | Symptom | Root cause | Fix | PR |
|----|---------|------------|-----|----|
| DEF-001 | Every booking was saved as **Cancha 5B**, whatever field was chosen | Ionic reused the confirm page: query params were read once and the `Idempotency-Key` was never renewed, so the API returned the first booking for the reused key | App: read params in `ionViewWillEnter`, new key per attempt. API: a key reused with a different payload answers `422 IDEMPOTENCY_KEY_REUSED` | pnk-app #6, pnk-api #2 |
| DEF-002 | The camera opened but the photo never appeared | Deprecated `Camera.getPhoto()` failed on the device and the adapter swallowed **every** error as "cancelled" | Camera 8 `takePhoto()` / `chooseFromGallery()`, resize to 1280 px in the adapter, real errors shown to the user | pnk-app #6 |
| DEF-003 | After sending the receipt the app offered to send it again | The app looked only at the booking status, not at the payment status; the detail page was also reused with stale data | Badge rules (`booking-badge.ts`), detail reloads on every entry. API rejects a second receipt in review (`409 RECEIPT_ALREADY_SENT`) | pnk-app #6, pnk-api #2 |
| DEF-004 | Booking detail showed "No hay conexión" offline although the list had the data | The detail only knew how to ask the API | Read-through cache of the detail plus fallback to the cached list row (stale-while-revalidate) | pnk-app #6 |
| DEF-005 | The queued receipt was sent but the screen stayed in *Sin señal*; a receipt could be dropped after 5 offline attempts | One flush attempt per "network back" event; offline attempts were counted as failures | Retry every 15 s while the queue is not empty and on app resume; offline never counts as a failure; screens notified through the `ReceiptSync` port | pnk-app #10 |

## Evidence

Screenshots of every case are in the final delivery (Anexo 1 — Fotografías).
