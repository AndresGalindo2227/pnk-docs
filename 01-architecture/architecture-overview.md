# Architecture Overview

Diagrams follow the [C4 model](https://c4model.com): each box is a person or a software element
and every arrow states **what** flows between them. The PlantUML sources are in [`diagrams/`](./diagrams/);
the Mermaid versions below render directly on GitHub.

## Level 1 — System context

```mermaid
flowchart LR
    player(["Player<br/>[Person]<br/>Books fields and pays deposits"])
    admin(["Field administrator<br/>[Person]<br/>Manages schedule and payments"])
    pnk["PANENKA Booking<br/>[Software system]<br/>App, portal and booking API"]
    invfox["InvFox<br/>[Existing system]<br/>Customers, sales, inventory"]
    wompi["Wompi<br/>[External system]<br/>Nequi, PSE and card payments"]

    player -->|"books fields with"| pnk
    admin -->|"manages bookings with"| pnk
    pnk -->|"reads and writes customers and sales in"| invfox
    pnk -->|"charges deposits through"| wompi
    wompi -->|"notifies payment results to (webhook)"| pnk
```

## Level 2 — Containers

```mermaid
flowchart TB
    player(["Player"])
    admin(["Field administrator"])

    subgraph PNK ["PANENKA Booking [software system]"]
        app["Mobile app / PWA — pnk-app<br/>[Ionic 9, Angular 22, Capacitor 8]<br/>Availability, booking, my bookings, payments"]
        cache[("Local cache<br/>[IndexedDB / Ionic Storage]<br/>Last fields and bookings")]
        vault[("Secure storage<br/>[Android Keystore]<br/>Access and refresh tokens")]
        portal["Admin portal — pnk-portal<br/>[Angular 22]<br/>Schedule, receipt approval, fields"]
        api["Booking API — pnk-api<br/>[PHP 8, REST/JSON, JWT]<br/>Business rules, authorization"]
        db[("Database — pnk-db<br/>[MySQL / MariaDB]<br/>InvFox schema + booking tables")]
    end

    invfoxweb["InvFox web<br/>[Existing PHP system]"]
    wompi["Wompi<br/>[Payment gateway]"]
    os["Android OS<br/>Camera, local notifications"]

    player --> app
    admin --> portal
    app -->|"HTTPS / JSON (OpenAPI contract)"| api
    portal -->|"HTTPS / JSON"| api
    app --- cache
    app --- vault
    app -->|"Capacitor plugins, permission just in time"| os
    api -->|"SQL (dedicated user, least privilege)"| db
    invfoxweb -->|"SQL (same schema)"| db
    api -->|"creates checkout"| wompi
    wompi -->|"signed webhook"| api
```

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
