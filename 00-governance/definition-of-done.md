# Definition of Done (DoD)

A user story is **done only when all of these are true**:

- [ ] Every acceptance criterion is met and verified on an **Android emulator or device**
      (evidence from `ionic serve` alone is not enough when the story uses a native capability).
- [ ] The code is merged through a Pull Request into `develop` (code repositories) or `main`
      (`pnk-docs`), and the PR closes the story (`Closes AndresGalindo2227/pnk-docs#N`).
- [ ] Unit tests cover the new use cases and state holders, and the whole suite passes.
- [ ] Lint passes, including the layer dependency rule (`presentation -> domain <- data`).
- [ ] API requests and responses match the OpenAPI contract.
- [ ] Every new endpoint checks **authorization on the server** (a user only reaches their own
      resources — see `security-rules.md`).
- [ ] Loading, empty, error and offline states are handled (and "permission denied" if applicable).
- [ ] No secrets, credentials or personal data were committed.
- [ ] Documentation is updated in `pnk-docs`: screen, endpoint, entity or ADR, whichever applies.
- [ ] The board card is moved to **Done**.

> A story that does not meet the full DoD is **not done** — it stays open. Partial work is not
> closed "to be finished later".
