# Definition of Ready (DoR)

A user story may move to **Ready** (and enter a sprint) **only when all of these are true**:

- [ ] The user value is written (As a / I want / So that).
- [ ] Acceptance criteria are written and verifiable (Given / When / Then).
- [ ] A prototype screen is linked, if the story touches the UI (app or portal).
- [ ] The endpoints involved are defined in the OpenAPI contract (`03-api-and-data/`),
      if the story calls the API.
- [ ] Loading, empty, error and offline states are described (plus "permission denied" when a
      native capability is used).
- [ ] The native permission is identified, if any (camera, notifications), and when it is requested.
- [ ] Dependencies on other stories are identified.
- [ ] `Priority`, `Estimate` and `Component` are set on the board.
- [ ] The estimate is **8 points or less** (split the story otherwise).

> If any box is unchecked, the story stays in Backlog — it does not enter the sprint.
