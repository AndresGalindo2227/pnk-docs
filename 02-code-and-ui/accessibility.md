# Accessibility

Accessibility is a **requirement**, not a nice-to-have. Target: **WCAG 2.2 level AA**. These rules
are checked before a story is Done.

## Rules

- Every interactive element has an accessible name: visible label, `aria-label` on icon-only
  buttons (e.g. `aria-label="Cancelar reserva"`), and `ion-input` with `label` / `labelPlacement`.
- Minimum touch target: **48×48 dp** on Android and **44×44 pt** on iOS. Slot chips in the
  availability grid are at least 48 px high.
- Text uses `rem` tokens and Ionic dynamic font scaling, so it grows with the OS font-size setting.
- Color contrast meets **WCAG AA**: 4.5:1 for body text, 3:1 for large text and icons. All color
  tokens in `design-system.md` were checked against white and the dark background.
- Meaning is never conveyed by color alone: slot status = color + text ("Libre" / "Ocupado"),
  errors = color + icon + message.
- Screens work with **TalkBack** (Android) and **VoiceOver** (iOS): headings use `ion-title` /
  `h1`–`h2`, lists use `ion-list`, and the result of an action is announced (`role="status"` for
  "Reserva creada").
- Respect "reduce motion": transitions use Ionic defaults, which honor `prefers-reduced-motion`.
- Focus order follows reading order; Ionic modals and alerts trap focus and restore it on close.
- Error messages appear next to the field and are linked with `aria-describedby`.

## Verification checklist (per screen)

- [ ] TalkBack pass on Android emulator for the primary flow (login → availability → booking).
- [ ] Largest OS font size does not break layout or hide the *Reservar* button.
- [ ] Contrast checked in light and dark mode.
- [ ] All actionable targets meet the minimum size.
- [ ] Reviewed in `md` and `ios` modes.

---

**Reference:** W3C. (2023). *Web Content Accessibility Guidelines (WCAG) 2.2*. https://www.w3.org/TR/WCAG22/
