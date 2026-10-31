# 03 — API & Data

> **What is this?** The rules for how PANENKA Booking talks to its backend (API) and how the app
> stores and syncs state locally (data). The single source of truth for the API is
> [`openapi.yaml`](./openapi.yaml) — **API First**: the contract is written and reviewed before
> `pnk-api`, `pnk-app` or `pnk-portal` implement it.

## Why this section exists

Networking and persistence are where most mobile bugs and security issues live: token
handling, retries, offline behavior, double bookings and schema migrations. Decide the rules
once and apply them in every feature's data layer.

---

## Documents in this section

| File | Purpose |
|------|---------|
| [openapi.yaml](./openapi.yaml) | OpenAPI 3.1 contract of `pnk-api` v1 (endpoints, schemas, errors) |
| [api-networking.md](./api-networking.md) | HTTP client, timeouts, retries, error mapping |
| [authentication.md](./authentication.md) | Token storage, refresh, sign-out |
| [persistence.md](./persistence.md) | Local storage rules and naming |
| [offline-sync.md](./offline-sync.md) | Offline strategy and conflict resolution |
| [_template-endpoint-integration.md](./_template-endpoint-integration.md) | Template to document consuming one endpoint |
| [_template-entity.md](./_template-entity.md) | Template to define a local persistence entity |

---

## Endpoints and user stories

All paths are relative to `/api/v1`.

| Method | Path | Story | Auth |
|--------|------|-------|------|
| GET | `/health` | — | Public |
| GET | `/config` | ADR-007 | Public |
| POST | `/auth/register` | HU-01 | Public |
| POST | `/auth/login` | HU-01 | Public |
| POST | `/auth/refresh` | HU-02 | Public (refresh token) |
| POST | `/auth/logout` | HU-02 | Player |
| GET | `/me` | HU-01 | Player |
| GET | `/fields` | HU-03 | Public |
| GET | `/availability?date=` | HU-03 | Public |
| GET | `/bookings?scope=` | HU-05 | Player |
| POST | `/bookings` | HU-04 | Player + `Idempotency-Key` |
| GET | `/bookings/{bookingId}` | HU-08 | Player (owner only, otherwise 404) |
| POST | `/bookings/{bookingId}/cancel` | HU-06 | Player (owner only) |
| POST | `/bookings/{bookingId}/payments/receipt` | HU-09 | Player (owner only) |
| POST | `/bookings/{bookingId}/payments/wompi` | HU-10 | Player (owner only) |
| POST | `/webhooks/wompi` | HU-10 | Wompi signature |
| GET | `/admin/schedule?date=` | HU-11 | Admin |
| GET | `/admin/payments?status=` | HU-11 | Admin |
| POST | `/admin/payments/{paymentId}/review` | HU-11 | Admin |

HU-07 (reminder) is local to the device and has no endpoint. HU-12 (weekly repeat) will be
added in contract v0.2 as an optional field of `POST /bookings` (additive change, ADR-007).

---

## Error catalog

Every error uses the same body: `{ "code": "...", "message": "...", "details": [...] }`.
The app maps **`code`**, never the `message`, to what the user sees.

| HTTP | Code | When |
|------|------|------|
| 400 | `INVALID_SIGNATURE` | Wompi webhook checksum does not match |
| 401 | `UNAUTHORIZED` | Missing or expired access token; the app refreshes once |
| 401 | `INVALID_CREDENTIALS` | Wrong email or password (never says which) |
| 401 | `SESSION_EXPIRED` | Refresh token expired or revoked; the app logs out |
| 403 | `FORBIDDEN` | Player calling an `/admin` endpoint |
| 404 | `NOT_FOUND` | Resource does not exist **or belongs to another player** |
| 409 | `EMAIL_ALREADY_REGISTERED` / `PHONE_ALREADY_REGISTERED` | Sign-up with existing data |
| 409 | `SLOT_ALREADY_TAKEN` | A sector of the field is held at that hour (ADR-004) |
| 409 | `CANCELLATION_WINDOW_CLOSED` | Less than `cancellationMinHours` before start |
| 409 | `BOOKING_NOT_CANCELLABLE` | Booking already cancelled or completed |
| 409 | `PAYMENT_ALREADY_REVIEWED` | Admin reviewing a payment twice |
| 413 | `FILE_TOO_LARGE` | Receipt photo over 2 MB |
| 422 | `VALIDATION_ERROR` | Invalid body or parameters; see `details` |
| 429 | `TOO_MANY_REQUESTS` | Too many login attempts; see `Retry-After` |
| 500 | `INTERNAL_ERROR` | Unexpected failure; never exposes stack traces |

---

## How to use the contract

```bash
# Validate the contract (Redocly CLI, uses redocly.yaml at the repo root)
npx @redocly/cli lint

# Run a mock server on http://localhost:4010/api/v1 (Prism)
npx @stoplight/prism-cli mock 03-api-and-data/openapi.yaml
```

The mock lets `pnk-app` be built before `pnk-api` exists. Contract changes go through a pull
request and bump `info.version` (SemVer).

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `01-architecture/` | Implements repository interfaces from the domain layer; ADR-004, ADR-005, ADR-007 |
| `00-governance/security-rules.md` | Token/transport rules are governed there |
| `04-quality/` | Contract tests and integration tests run against this file |
