# 02 — Code & UI

> **What is this?** The coding standards and the UI rules of **PANENKA Booking** (`pnk-app` and
> `pnk-portal`) — how code is written and how the interface is built, styled, made accessible and
> worded.

## Why this section exists

Consistency at the code and UI level is what makes a mobile app feel like one product. These rules
are enforced in review (Definition of Done) and by ESLint, Prettier and Stylelint.

---

## Documents in this section

| File | Purpose |
|------|---------|
| [coding-standards.md](./coding-standards.md) | Naming, immutability, error handling, async rules (Angular + RxJS + Signals) |
| [design-system.md](./design-system.md) | PANENKA tokens on top of Ionic CSS variables, `md` / `ios` modes, components |
| [accessibility.md](./accessibility.md) | WCAG 2.2 AA rules for Ionic screens |
| [i18n.md](./i18n.md) | Centralized Spanish texts, error-code messages, COP and date formatting |
| [_template-component.md](./_template-component.md) | Template to document a shared UI component |

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `01-architecture/` | Standards apply within the layers defined in `layers-and-state.md` |
| `03-api-and-data/` | API error codes are translated to user texts here (`i18n.md`) |
| `04-quality/` | Lint and accessibility rules are checked before a story is Done |

---

## Answers to the questions of this section

| Question | Answer |
|----------|--------|
| How is code named, structured and error-handled? | [coding-standards.md](./coding-standards.md) |
| Where do colors, spacing and typography come from? | `src/theme/variables.scss` + `src/theme/tokens.scss` — [design-system.md](./design-system.md) |
| How do we guarantee the app is accessible? | Checklist per screen — [accessibility.md](./accessibility.md) |
| How is user-facing text centralized? | `src/app/core/i18n/texts.es.ts` — [i18n.md](./i18n.md) |
