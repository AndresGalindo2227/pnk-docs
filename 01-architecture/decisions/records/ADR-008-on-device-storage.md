# ADR-008 — On-device storage: secure storage, Preferences and no local database

- **ID:** ADR-008
- **Date:** 2026-10-31
- **Status:** Accepted
- **Authors:** Andrés Felipe Galindo Dávila

---

## Context

The app must keep login tokens safely, show *Mis reservas* without connection (ADR-004) and queue
receipt photos taken offline. The framework asks to choose **one** local store and record it.

**Known constraints:**
- Tokens must never be readable by other apps or from a device backup in plain text.
- The cached data is small: the upcoming bookings of one player (usually fewer than 20).
- The same code also runs as a PWA in the browser (ADR-006).

---

## Decision

**We decided:**
- **Tokens** → `@aparajita/capacitor-secure-storage` (Android Keystore / iOS Keychain), behind the
  `TokenStore` port. On the PWA the refresh token is not persisted.
- **Cache and queue index** → `@capacitor/preferences` (key-value, JSON values), behind repository ports.
- **Queued receipt photos** → `@capacitor/filesystem`, app data directory.
- **No SQLite in V1.**

**Justification:** the data volume and query needs are tiny, so a key-value store is enough and
keeps the app simple; secrets get the platform's hardware-backed storage, as required by OWASP MASVS.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|-------------|------|------|-----------------------|
| Secure storage + Preferences + Filesystem (chosen) | Simple, right tool per data type | Two plugins to wrap | — (chosen) |
| Everything in Preferences | One plugin | Tokens in plain storage | Violates `security-rules.md` |
| SQLite (`@capacitor-community/sqlite`) | Queries, transactions | Schema and migrations for < 20 rows | Overkill for V1 |
| `localStorage` / IndexedDB | Works on web | Not secure; can be cleared by the OS on iOS | Insecure and unreliable |
| Ionic Identity Vault | Biometrics, enterprise support | Paid license | Not justified for V1 |

---

## Consequences

**Positive:** tokens protected by the OS keystore; cache rebuilt from the API at any time.

**Negative / Trade-offs:** PWA users log in again after closing the tab; no offline search.

**Impact on the system:** `core/storage/`, features `auth`, `my-bookings`, `payments`.
Documents: `03-api-and-data/authentication.md`, `persistence.md`, `offline-sync.md`.
