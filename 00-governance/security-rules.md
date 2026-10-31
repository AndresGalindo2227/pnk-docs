# Security Rules

Non-negotiable rules for the PANENKA Booking app, portal and API.

## Secrets and credentials

- Never commit API keys, tokens, passwords or signing keys (see `git-conventions.md`, *Never committed*).
- Each installation keeps its secrets in files ignored by Git (`.env`, `config/config.php`);
  the repository only contains `*.example` templates.
- CI pipelines read secrets from GitHub Actions secrets.

## Authentication

- Players and staff authenticate with a short-lived **JWT access token (15 minutes)** and a
  **refresh token (7 days)**.
- Refresh tokens are stored in the database **hashed (SHA-256)**, can be revoked, and are rotated on
  every refresh.
- Passwords are stored with `password_hash()` (bcrypt); plain passwords are never stored or logged.
- On the device, tokens live only in **secure storage** (Android Keystore). Never in
  `localStorage`, `Preferences` or plain files.

## Authorization (resource level)

- Being authenticated is not enough: **every request checks that the user may access that specific
  resource**. A player only reads or changes their own bookings, payments and profile.
- Queries are always filtered by the authenticated subject (e.g. `WHERE player_id = :currentPlayer`),
  never only by the id received in the URL.
- When a resource exists but belongs to someone else, the API answers **404**, not 403, so it does
  not reveal which ids exist.
- Administrator endpoints require the `admin` role in the token and are re-checked on the server.

## Transport

- All traffic over HTTPS in every non-local environment. No cleartext HTTP in release builds.
- Certificate pinning for the auth and payment endpoints is evaluated for V2 (recorded as a
  pending decision).

## Payments

- The Wompi webhook is accepted only when its signature is valid; card data never touches our servers.
- A booking changes to "confirmed" only from the server-side webhook or an administrator approval,
  never from a value sent by the app.

## Input and data

- Validate every input on the server (types, ranges, dates in the future, field exists), even if the
  app already validated it.
- All SQL uses prepared statements with bound parameters.
- Escape every value printed into HTML (portal and InvFox views).
- Do not cache personal data that the feature does not need; clear the local cache on logout.
- Logs never contain passwords, tokens or full documents of customers.

## Database

- The application connects with a dedicated user that has only `SELECT, INSERT, UPDATE, DELETE` on its
  database. Structure changes run as a separate administrative step (migrations in `pnk-db`).

## Mobile permissions

- Request the minimum permissions, **just in time**, with a clear rationale shown before the
  system dialog: notifications when the first booking is confirmed, camera when the player
  photographs a receipt.
- The app keeps working when a permission is denied (the booking is saved without a reminder; the
  receipt can be picked from the gallery).

## Build hardening

- Release builds are minified and have debug logs and debugging flags disabled.

## Handling vulnerabilities

- Report suspected vulnerabilities privately to the tech lead, never in a public issue.
- Run `npm audit` and `composer audit` at least once per sprint; patch known-vulnerable
  dependencies within one sprint of disclosure.
