# Offline & Sync

Decided in [ADR-004](../01-architecture/decisions/records/ADR-004-booking-consistency-and-offline-strategy.md):
**writes that can conflict are online-only; reads are cached; uploads that cannot conflict are queued.**

## Per-feature strategy

| Feature | Strategy | Offline behavior |
|---------|----------|------------------|
| Sign-up / login (HU-01, HU-02) | Online-only | Button disabled, banner "Sin conexión" |
| Availability (HU-03) | Online-only | Offline state; availability is never shown from cache because it changes every minute |
| Create / cancel booking (HU-04, HU-06) | Online-only | Button disabled; nothing is queued, so a slot is never promised offline |
| My bookings (HU-05) | **Read-through cache** | Last saved list with "Actualizado hace X min" |
| Booking reminder (HU-07) | Local only | Scheduled on the device after the booking is created; works offline |
| Receipt photo (HU-09) | **Offline queue** | Saved locally, uploaded automatically when the connection returns |

## Read-through cache (My bookings)

1. Show the cached list immediately if it exists (state `loaded`, flag `isStale`).
2. Request `GET /bookings`; on success replace the cache and the screen.
3. On network error keep the cached list and show the stale banner; with no cache, show the offline
   state with *Reintentar*.

## Receipt upload queue

- Each queued item has a client-generated `id` and is uploaded with
  `POST /bookings/{id}/payments/receipt`.
- **Conflict policy: server-wins.** If the booking was cancelled or the payment already reviewed
  (`404` / `409`), the item is discarded and the player is notified.
- Replaying is safe: the API answers `409 PAYMENT_ALREADY_REVIEWED` for duplicates, so an item is
  never applied twice.
- Retries use backoff (30 s, 2 min, 10 min) and stop after 5 attempts; the booking detail shows a
  manual *Reintentar envío* action.

## Connectivity

- Connectivity is a **stream**: `@capacitor/network` `networkStatusChange` feeds a
  `ConnectivityService` signal used by stores and the global offline banner.
- When the device reconnects: refresh the visible screen and flush the receipt queue.

## Checklist

- [x] Strategy documented per feature.
- [x] Conflict policy defined for the only queued entity (receipt: server-wins).
- [x] Queue replay is idempotent.
- [x] UI shows offline and stale states (`pnk-state-view`, banner).
