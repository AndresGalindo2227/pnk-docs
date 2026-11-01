# Persistence (on-device data)

## Choice

PANENKA Booking does **not** need a local database: it caches a short list and queues a few
uploads. V1 uses two key-value stores behind repository ports (ADR-008):

| Store | Plugin | Holds |
|-------|--------|-------|
| Secure storage | `@aparajita/capacitor-secure-storage` (Keystore / Keychain) | Access and refresh tokens |
| Preferences | `@capacitor/preferences` | Cached *Mis reservas*, last availability date viewed, receipt upload queue index |
| Filesystem | `@capacitor/filesystem` (app data directory) | Receipt photos waiting to be uploaded |

SQLite is not used in V1. It would be reconsidered (new ADR) if the app ever needed offline
queries or large data sets.

## Keys

| Key | Type | Content |
|-----|------|---------|
| `pnk.cache.myBookings.v1` | JSON | `{ savedAt, items: BookingCacheItem[] }` (upcoming bookings only) |
| `pnk.prefs.lastDate.v1` | string | `YYYY-MM-DD` |
| `pnk.queue.receipts.v1` | JSON | `[{ id, bookingId, amount, filePath, createdAt, attempts }]` |

- Keys are namespaced `pnk.<area>.<name>.v<version>`.
- Cached models (`BookingCacheItem`) are **separate** from domain entities; the data layer maps them.

## Rules

- Store only what the app needs offline: upcoming bookings of the logged-in player, never other
  players' data, never payment card data.
- All storage access goes through the data layer; pages never call Preferences directly.
- Everything except the last viewed date is deleted on sign-out.
- Server data (bookings, payments) lives in MySQL; its migrations are in
  [`pnk-db`](https://github.com/AndresGalindo2227/pnk-db).

## Versioning of cached data

- A structural change of a cached JSON bumps the key version (`.v1` → `.v2`). On start, the app
  deletes unknown or old versions instead of trying to read them — the cache is rebuilt from the API.
