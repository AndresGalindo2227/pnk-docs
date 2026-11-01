# API / Networking

How `pnk-app` and `pnk-portal` call `pnk-api`. The contract is [`openapi.yaml`](./openapi.yaml).

## Client

- **One** Angular `HttpClient`, configured once in `app.config.ts` with
  `provideHttpClient(withInterceptors([...]))`. Repositories in each feature's `data/` folder inject
  it; no feature creates its own client or uses `fetch`.
- The base URL comes from `environment.apiBaseUrl` (`/api/v1`, see ADR-007).

## Interceptors (in `core/network/`, in this order)

| # | Interceptor | Responsibility |
|---|-------------|----------------|
| 1 | `headersInterceptor` | Adds `X-App-Version`, `X-Request-Id` (UUID) and `Accept: application/json` |
| 2 | `authInterceptor` | Adds `Authorization: Bearer <access token>`; on `401`, refreshes **once** and retries the original request (see `authentication.md`) |
| 3 | `timeoutInterceptor` | RxJS `timeout(15000)` on every request; uploads use `60000` |
| 4 | `errorInterceptor` | Maps `HttpErrorResponse` to `AppError` (see `02-code-and-ui/coding-standards.md`) |

## Requests & retries

- **Automatic retries only on GET**, with RxJS `retry({ count: 2, delay: backoff })` and exponential
  backoff (1 s, 2 s) for network errors, `5xx` and timeouts.
- **Never auto-retry POST.** `POST /bookings` carries an `Idempotency-Key` generated once per
  attempt; if the user taps *Reintentar*, the same key is reused, so the API never creates two
  bookings.
- `409`, `404`, `422` are final answers — no retry; they are shown to the user.

## Serialization

- JSON is read into **DTOs** (`booking.dto.ts`) that mirror the OpenAPI schemas, then mapped to
  domain entities (`toBooking(dto)`) in the data layer.
- The UI never sees a raw payload. Dates arrive as ISO 8601 with offset and become `Date` objects
  in the mapper; money stays an integer of COP.
- Malformed payloads raise `AppError { kind: 'unknown' }`; the mapper does not guess.

## Status handling

| Status | Meaning | App behavior |
|--------|---------|--------------|
| 2xx | Success | Map to domain model |
| 401 `UNAUTHORIZED` | Access token expired | Refresh once and retry; if refresh fails → sign out |
| 404 `NOT_FOUND` | Missing or another player's resource | "No encontramos esta reserva" + back to *Mis reservas* |
| 409 `SLOT_ALREADY_TAKEN` | Someone else booked first | Message + reload availability of that day |
| 422 `VALIDATION_ERROR` | Invalid input | Show `details` next to each field |
| 5xx / timeout / no network | Transient | GET: backoff + retry; then error state with *Reintentar* |

## Logging

- Never log tokens, passwords, full `Authorization` headers or receipt images.
- Production builds disable `console.debug`; errors are logged with `X-Request-Id` only, so they
  can be matched with `pnk-api` logs without exposing personal data.
