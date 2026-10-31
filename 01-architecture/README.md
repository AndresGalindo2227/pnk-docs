# 01 — Architecture

> How PANENKA Booking is organized: system context and containers (C4), folder structure,
> layers and state, navigation, and the record of architectural decisions (ADRs).

## Documents in this section

| File | Purpose |
|------|---------|
| [architecture-overview.md](./architecture-overview.md) | C4 context and container diagrams, booking model (field sectors) |
| [project-structure.md](./project-structure.md) | Feature-first folder layout of `pnk-app` and module rules |
| [layers-and-state.md](./layers-and-state.md) | Presentation / Domain / Data layers, dependency rule and state pattern |
| [navigation.md](./navigation.md) | Route table of the app and guard rules |
| [diagrams/](./diagrams/) | C4 and layer diagrams as PlantUML sources (`.puml`) |
| [decisions/](./decisions/) | Architecture Decision Records (ADRs) |
| [_template-feature.md](./_template-feature.md) | Template to document one feature end to end |
| [_template-screen.md](./_template-screen.md) | Template to spec a single screen |

## Technology stack (V1)

| Part | Technology |
|------|------------|
| Mobile app (`pnk-app`) | Ionic 9 + Angular 22 (standalone, signals) + Capacitor 8, Android first |
| Player web version | Same `pnk-app` code built as a PWA (ADR-006) |
| Admin portal (`pnk-portal`) | Angular 22 web application (ADR-006) |
| API (`pnk-api`) | PHP 8 REST API, module integrated with InvFox, JWT + refresh tokens (ADR-005) |
| Database (`pnk-db`) | MySQL / MariaDB, shared InvFox schema plus booking tables, versioned migrations |
| Infrastructure (`pnk-infra`) | Docker Compose: database, API, portal and app (web build) |

## Questions this section answers

- What are the parts of the system and how do they talk to each other?
- Where does a new feature's code go, and which layer may depend on which?
- What single state pattern does the app use?
- How do screens navigate and how are routes protected?
- Which decisions were taken, with which alternatives, and why?
