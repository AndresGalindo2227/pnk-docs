# Design System

PANENKA Booking uses **Ionic components** themed through **Ionic CSS variables**. Our own design
tokens are a thin layer on top of those variables, so every Ionic component picks up the brand
automatically and screens never contain raw colors or sizes.

## Where values come from

| File | Content |
|------|---------|
| `src/theme/variables.scss` | Ionic variables: `--ion-color-primary`, `--ion-background-color`, `--ion-font-family`, dark palette |
| `src/theme/tokens.scss` | PANENKA tokens (`--pnk-*`) for spacing, radius, typography and status colors |
| `src/global.scss` | Imports both files; no component styles |

## Tokens

### Color — mapped to Ionic variables

| Token / Ionic variable | Light | Use |
|------------------------|-------|-----|
| `--ion-color-primary` | `#1B7F3B` (field green) | Main actions: *Reservar*, *Confirmar* |
| `--ion-color-secondary` | `#0F3D23` | Headers, selected tab |
| `--ion-color-tertiary` | `#F2B705` | Highlights (price, "hoy") |
| `--ion-color-success` | `#2E7D32` | Slot available, booking confirmed |
| `--ion-color-warning` | `#B26A00` | Pending payment |
| `--ion-color-danger` | `#C62828` | Slot taken, errors, cancel |
| `--ion-color-medium` | `#5F6B66` | Secondary text |

Each color also defines its `-rgb`, `-contrast`, `-shade` and `-tint` variants, generated with the
Ionic color generator, so `color="primary"` works on every component.

### Spacing, radius and typography — `--pnk-*`

| Group | Tokens |
|-------|--------|
| Spacing (4-pt scale) | `--pnk-space-xs: 4px` · `sm: 8px` · `md: 16px` · `lg: 24px` · `xl: 32px` |
| Radius | `--pnk-radius-sm: 8px` · `md: 12px` · `pill: 999px` |
| Typography | `--pnk-text-title: 1.375rem` · `body: 1rem` · `caption: 0.875rem` (rem, so text scales) |
| Slot status | `--pnk-slot-free: var(--ion-color-success)` · `--pnk-slot-taken: var(--ion-color-danger)` |

## Platform modes: `md` and `ios`

- Ionic renders **Material Design (`md`) on Android** and **iOS style (`ios`) on iPhone and in
  Safari**. We keep the automatic mode; it is never forced globally.
- Mode-specific differences (header shadow, back button text, segment style) are left to Ionic.
  Only when a token must differ we use `.md` / `.ios` selectors in `tokens.scss`, never in pages.
- The PWA in a desktop browser renders `md`.
- Screens are reviewed in both modes with `ionic serve` and the `?ionic:mode=ios` URL parameter.

## Dark mode

Light and dark themes share the **same token names**; only values change inside the
`@media (prefers-color-scheme: dark)` block of `variables.scss` (Ionic "system" dark palette).

## Rules

- No hardcoded colors, sizes or font sizes in pages or components — use tokens. Stylelint blocks
  hex values outside `src/theme/`.
- Build screens from Ionic components and our `shared/` components — no one-off styled widgets.
- Respect safe areas (`ion-header`, `ion-content`) and the Android hardware back button.
- Status is never shown by color alone (see `accessibility.md`): a taken slot is red **and** says
  "Ocupado".

## Shared components (V1)

| Component | Purpose | States |
|-----------|---------|--------|
| `pnk-slot-chip` | One hour of one field in the availability grid | free / taken / selected / past |
| `pnk-booking-card` | Booking summary in *Mis reservas* | pending / confirmed / cancelled |
| `pnk-state-view` | Shared loading / empty / error / offline view with retry action | 4 states |
| `pnk-price` | COP amount formatted per `i18n.md` | — |

Each one is documented with [`_template-component.md`](./_template-component.md) when created.
