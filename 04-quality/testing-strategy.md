# Testing Strategy

## Test Pyramid

| Type | Scope | Speed | Share |
|------|-------|-------|-------|
| Unit | Use cases, state holders, mappers — no UI, no I/O | Fast | ~70% |
| Widget / UI | A single screen/component in isolation | Medium | ~20% |
| Integration / E2E | Full flow on a simulator/emulator or device | Slow | ~10% |

## Rules

- Every use case and state holder has unit tests; mock only **direct** dependencies.
- Test **behavior and state transitions**, not implementation details.
- Keep tests deterministic: no real network, no wall-clock sleeps, fixed clock/seed.
- Arrange-Act-Assert; one logical assertion per test where possible.

## Naming

- `<unit>_<condition>_<expectedResult>`
  - Example: `login_whenTokenExpired_showsSessionError`

## Coverage

- **80% minimum** on `domain/` (use cases) and state holders.
- Coverage is a floor, not a goal — a covered line is not a tested behavior.

## CI

- Unit + widget suites run on **every PR**; the PR cannot merge if they fail.
- Integration/E2E run on a nightly and pre-release job on at least one iOS and one Android device/emulator.

---

## Applied in PANENKA Booking (v0.1.0)

### Tools

| Level | Tool | Where |
|-------|------|-------|
| Unit (use cases, state holders, mappers) | **Vitest** through the Angular `unit-test` builder | `pnk-app/src/**/*.spec.ts` |
| Test doubles | Hand-written fakes (no mocking library) | `pnk-app/src/testing/` |
| API rules | Plain PHP test runner | `pnk-api/tests/run.php` |
| End-to-end on a real device | Manual test cases on a HONOR RMO-NX3 (Android) | [`device-test-report.md`](./device-test-report.md) |

- Fakes implement the **domain ports** (`FakeBookingsRepository`, `FakeReceiptsRemote`,
  `InMemoryReceiptQueue`, `FakeCapturePort`, `FixedClock`…). Because pages and stores only depend on
  ports, no test needs a network, a database, a camera or the real clock.
- Stores are tested through `TestBed` with the fakes provided in place of the adapters.

### Suites (42 automated tests, all green in CI)

| Suite | What it proves |
|-------|----------------|
| `sign-up.usecase` | Phone and password rules of HU-01 |
| `availability` / `availability.store` | Sector rule (Cancha 9 = 5A + 5B + 5C), past slots, offline message |
| `create-booking.usecase` | Hours 1–3, slot in the past, same Idempotency-Key on retry |
| `schedule-reminder.usecase` | Reminder 60 min before kickoff, permission denied, past reminders skipped |
| `load-my-bookings.usecase` / `my-bookings.store` | Read-through cache: fresh, stale with age, offline without copy |
| `upload-receipt.usecase` | Online upload, offline queue, server-wins, **never drop a receipt while offline**, give up after 5 server errors |
| `receipt-upload.store` | Preview, permission denied, sending, queued → uploaded by itself after the flush |
| `booking-badge` | *Pendiente de pago* / *Pago en revisión* / *Comprobante rechazado* rules |

### Gate

- `npm run lint && npm run test:unit` must be green before every commit (Definition of Done).
- The same commands run in **GitHub Actions on every pull request** and on every push to
  `develop`, `qa` and `main` (see [`05-release/ci-cd.md`](../05-release/ci-cd.md)). A red check
  blocks the merge.
- E2E automation with Playwright is configured (`pnk-app/e2e/`) but not part of the gate yet; the
  device flows are covered by the manual cases of the device test report.
