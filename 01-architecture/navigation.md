# Navigation

## Model

- All routes live in **one declarative table**: `src/app/app.routes.ts`
  (see [ADR-002](./decisions/records/ADR-002-navigation-route-table-and-guards.md)).
- Pages are lazy-loaded with `loadComponent`.
- Protected routes are guarded **in the table** with functional guards (`authGuard`, `guestGuard`),
  never inside a page's `ngOnInit`.
- Routes receive **identifiers**, not objects; the destination page loads the data it needs.
- Static segments go before parameters (`bookings/new` before `bookings/:id`).
- Hardware back on Android follows the Ionic navigation stack.
- Every authenticated route is a child of the **tabs shell** (`core/shell/shell.page`), which shows
  the bottom tabs *Reservar*, *Mis reservas* and *Perfil*. Feature providers are attached at route
  level (`provideBookings()`, `provideMyBookings()`, `providePayments()`), so each feature is
  composed only where it is used.
- **Ionic keeps pages alive and reuses them.** Data that depends on route parameters is read in
  `ionViewWillEnter`, never once in the constructor or `ngOnInit` (defect DEF-001 in
  `04-quality/device-test-report.md`).

## Route registry — `pnk-app`

| Route | Page | Args | Guard | Story |
|-------|------|------|-------|-------|
| `/login` | `LoginPage` | — | `guestGuard` | HU-02 |
| `/sign-up` | `SignUpPage` | — | `guestGuard` | HU-01 |
| `/availability` | `AvailabilityPage` | query `date` | `authGuard` | HU-03 |
| `/bookings/new` | `BookingConfirmPage` | query `fieldId`, `start` | `authGuard` | HU-04, HU-07, HU-10 |
| `/bookings/:id/receipt` | `ReceiptUploadPage` | `id`, query `amount` | `authGuard` | HU-09 |
| `/bookings/:id` | `BookingDetailPage` | `id` | `authGuard` | HU-06, HU-09 |
| `/my-bookings` | `MyBookingsPage` | — | `authGuard` | HU-05 |
| `/profile` | `ProfilePage` | — | `authGuard` | — |
| `**` | redirect to `/availability` | — | — | — |

```ts
export const routes: Routes = [
  {
    path: 'login',
    canMatch: [guestGuard],
    providers: [provideAuth()],
    loadComponent: () => import('./features/auth/presentation/login.page'),
  },
  {
    path: 'sign-up',
    canMatch: [guestGuard],
    providers: [provideAuth()],
    loadComponent: () => import('./features/auth/presentation/sign-up.page'),
  },
  {
    path: '',
    canMatch: [authGuard],
    loadComponent: () => import('./core/shell/shell.page'),
    providers: [provideBookings(), provideMyBookings()],
    children: [
      { path: 'availability', loadComponent: () => import('./features/bookings/presentation/availability.page') },
      // Static segment before the parameter: 'bookings/new' must not match 'bookings/:id'.
      { path: 'bookings/new', loadComponent: () => import('./features/bookings/presentation/booking-confirm.page') },
      {
        path: 'bookings/:id/receipt',
        providers: [providePayments()],
        loadComponent: () => import('./features/payments/presentation/receipt-upload.page'),
      },
      { path: 'bookings/:id', loadComponent: () => import('./features/bookings/presentation/booking-detail.page') },
      { path: 'my-bookings', loadComponent: () => import('./features/my-bookings/presentation/my-bookings.page') },
      { path: 'profile', loadComponent: () => import('./features/auth/presentation/profile.page') },
      { path: '', pathMatch: 'full', redirectTo: 'availability' },
    ],
  },
  { path: '**', redirectTo: 'availability' },
];
```

> This table is the single source of truth for the screens of the app. Any change to
> `app.routes.ts` updates this file in the same pull request.

The route table of the admin portal (`/login`, `/schedule`, `/receipts`, `/fields`) is documented in
`pnk-portal` and protected by an `adminGuard` that requires the `admin` role.
