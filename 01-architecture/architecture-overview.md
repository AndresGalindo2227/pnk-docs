# Architecture Overview

Diagrams follow the [C4 model](https://c4model.com): each box is a person or a software element
and every arrow states **what** flows between them. The Mermaid diagrams below render directly on
GitHub and describe **what is built in v0.1.0**. Elements drawn with a dashed border are
**planned (v0.2.0)** and are not part of the current release. The PlantUML sources in
[`diagrams/`](./diagrams/) show the full target architecture.

## Level 1 — System context

```mermaid
flowchart LR
    player(["Player<br/>[Person]<br/>Books fields, uploads the deposit receipt"])
    admin(["Field administrator<br/>[Person]<br/>Approves receipts, manages the schedule"])
    pnk["PANENKA Booking<br/>[Software system]<br/>Mobile app, PWA and booking API"]
    invfox["InvFox<br/>[Existing system]<br/>Customers, sales, inventory"]
    wompi["Wompi<br/>[External system — planned v0.2.0]<br/>Online payments"]

    player -->|"books fields and sends receipts with"| pnk
    admin -->|"reviews receipts and bookings in"| invfox
    pnk -->|"reads and writes customers and bookings in (same database)"| invfox
    pnk -.->|"planned: charges deposits through"| wompi

    classDef planned stroke-dasharray: 5 5,color:#666
    class wompi planned
```

In v0.1.0 the deposit is paid by **Nequi transfer**: the player uploads a photo of the receipt and the
administrator approves it. Online payment (Wompi) is deferred — see
[ADR-006](./decisions/records/ADR-006-web-channels-pwa-and-admin-portal.md) and
[technical debt](../04-quality/technical-debt.md).

## Level 2 — Containers

```mermaid
flowchart TB
    player(["Player"])
    admin(["Field administrator"])

    subgraph PNK ["PANENKA Booking [software system]"]
        app["Mobile app / PWA — pnk-app<br/>[Ionic 9, Angular 22, Capacitor 8]<br/>Availability, booking, my bookings, receipts"]
        prefs[("Local cache + receipt queue<br/>[Capacitor Preferences]<br/>My bookings, booking detail, pending photos")]
        vault[("Secure storage<br/>[Android Keystore — memory only on the PWA]<br/>Access and refresh tokens")]
        api["Booking API — pnk-api<br/>[PHP 8, REST/JSON, JWT]<br/>Business rules, authorization, idempotency"]
        db[("Database — pnk-db<br/>[MySQL / MariaDB]<br/>InvFox schema + booking tables")]
        portal["Admin portal — pnk-portal<br/>[Angular 22 — planned v0.2.0]"]
    end

    invfoxweb["InvFox web<br/>[Existing PHP system]"]
    os["Android OS<br/>Camera, gallery, local notifications"]

    player --> app
    admin --> invfoxweb
    admin -.-> portal
    app -->|"HTTPS / JSON (OpenAPI contract), Bearer JWT, Idempotency-Key"| api
    portal -.->|"planned: HTTPS / JSON"| api
    app --- prefs
    app --- vault
    app -->|"Capacitor plugins, permission requested just in time"| os
    api -->|"SQL (dedicated user, least privilege)"| db
    invfoxweb -->|"SQL (same schema)"| db

    classDef planned stroke-dasharray: 5 5,color:#666
    class portal planned
```

Storage decisions: [ADR-008](./decisions/records/ADR-008-on-device-storage.md).
Offline behaviour: [offline-sync](../03-api-and-data/offline-sync.md).

## Deployment — v0.1.0

```mermaid
flowchart LR
    subgraph phone ["Android phone"]
        apk["pnk-app<br/>signed release APK"]
    end
    subgraph browser ["Any browser"]
        pwa["pnk-app PWA"]
    end
    subgraph firebase ["Firebase Hosting"]
        web["panenka-booking.web.app<br/>static build"]
    end
    subgraph ngrok ["ngrok (static HTTPS domain)"]
        tunnel["hangout-frown-salt.ngrok-free.dev"]
    end
    subgraph pc ["Developer PC (local server)"]
        php["pnk-api<br/>PHP 8 :8000"]
        mysql[("MySQL<br/>InvFox + booking tables")]
    end

    web -->|"serves"| pwa
    apk -->|"HTTPS"| tunnel
    pwa -->|"HTTPS (CORS allow-list)"| tunnel
    tunnel -->|"forwards"| php
    php -->|"SQL"| mysql
```

Why this set-up and what replaces it in production:
[ADR-010](./decisions/records/ADR-010-deployment-https-tunnel-firebase.md) and
[deployment](../05-release/deployment.md).

## Booking model — field sectors

PANENKA has **one soccer-9 pitch that can be split into three soccer-5 fields**. The system
separates *what is booked* from *the physical space it occupies*:

| Bookable field | Price per hour | Occupies sectors |
|----------------|----------------|------------------|
| Cancha 5A | $70.000 | A |
| Cancha 5B | $70.000 | B |
| Cancha 5C | $70.000 | C |
| Cancha 9 | $210.000 | A + B + C |

A booking holds its sectors for every hour it lasts (`booking_slots`, primary key
`(unit_id, slot_start)`). Because a sector can be held only once per hour, the **database itself**
rejects any overlap:

| Already booked at 18:00 | Can also be booked at 18:00 |
|-------------------------|-----------------------------|
| Cancha 5A | 5B, 5C — **not** Cancha 9 |
| Cancha 9 | none |
| 5A and 5B | 5C — **not** Cancha 9 |

See [ADR-004](./decisions/records/ADR-004-booking-consistency-and-offline-strategy.md) and the
`pnk-db` migrations.
