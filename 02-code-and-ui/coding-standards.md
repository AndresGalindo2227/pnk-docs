# Coding Standards

Applies to `pnk-app` and `pnk-portal` (Angular 22, TypeScript strict). `pnk-api` (PHP 8) follows
PSR-12 and documents its own rules in its repository.

## Principles

- **Immutability first** — `readonly` properties, `const`, and new objects instead of mutation.
  Store state is replaced, never edited in place (see `01-architecture/layers-and-state.md`).
- **Small units** — a function does one thing; a file declares one primary type.
- **No magic values** — business numbers come from `GET /config` (cancellation hours, deposit
  percent, opening hours); visual values come from design tokens.
- **Fail fast** — validate inputs at boundaries (forms, DTO mappers); never swallow errors.
- **TypeScript strict** — `strict: true`, no `any` (use `unknown` and narrow it).

## Naming

| Kind | Convention | Example |
|------|------------|---------|
| Types / classes / interfaces | `PascalCase` | `Booking`, `BookingsRepository` |
| Members / variables / signals | `camelCase` | `selectedDate`, `state` |
| Constants | `UPPER_SNAKE_CASE` | `REQUEST_TIMEOUT_MS` |
| Observables | `camelCase` + `$` | `bookings$` |
| Files | `kebab-case.role.ts` | `create-booking.usecase.ts` |
| Booleans | `is` / `has` / `can` / `should` | `canCancel`, `isOffline` |
| Language | English for all code, comments and commits | — |

## Error handling

- The data layer maps HTTP errors to a typed **`AppError`** union:
  `{ kind: 'network' } | { kind: 'unauthorized' } | { kind: 'not-found' } |
  { kind: 'conflict', code: string } | { kind: 'validation', fields: string[] } | { kind: 'unknown' }`.
- Use cases return results or throw `AppError`; pages never read `HttpErrorResponse`.
- Every user-facing failure has a message (from `i18n.md`) **and** a recovery action:
  *Reintentar*, *Elegir otra hora*, *Iniciar sesión*.
- The API `code` (e.g. `SLOT_ALREADY_TAKEN`) decides the message, never the API `message` text.

## Async

- HTTP calls are RxJS observables; stores convert results to signals.
- Always handle the error branch (`catchError` in the repository, `error` state in the store).
- Cancel in-flight work when a page is destroyed (`takeUntilDestroyed()`).
- Never block the UI: images (receipt photos) are compressed with the Capacitor Camera quality
  option before upload, not in a loop on the main thread.

## Linting and formatting

- ESLint (`angular-eslint`) with `no-restricted-imports` enforcing `presentation → domain ← data`.
- Prettier for formatting, Stylelint for SCSS (blocks raw hex colors outside `src/theme/`).
- **CI fails on warnings.** An inline `eslint-disable` needs a comment explaining why.
