# Solo Feature Workflow

Ship one useful, coherent change at a time. Keep requirements small and put financial behavior under executable tests.

## Sources of truth

The living references below are the active baseline; archived specifications preserve historical context. If sources conflict, identify the conflict and resolve it explicitly; do not guess at financial behavior. Prototypes illustrate UI states and do not override financial rules.

The active references are `V1_SPEC.md` for scope and flows, `ARCHITECTURE.md` for modules and data integrity, `FINANCIAL_RULES.md` for financial definitions and calculations, `TEST_SCENARIOS.md` for cases guiding tests, `DECISIONS.md` for consequential choices and open questions, and `docs/design/assets/` for visual references. `docs/archive/` preserves historical material.

## The feature loop

Use these four steps for each feature, fix or behavior change. A small change can fit in a few lines. A feature brief fits on one page and can live in an issue, temporary note or change description; it does not need a permanent file.

### 1. Problem and scope check

Write down:

- **User state:** What situation is the user in, and what can they not do or understand?
- **Outcome:** What observable change solves it?
- **In scope:** What is the smallest useful behavior?
- **Out of scope:** What will this slice explicitly not do?
- **Done:** How will the behavior and relevant UI state be demonstrated?

Check v1 scope and the closest prototype. Make scope changes explicitly; a prototype alone does not expand scope. If the outcome is too large for one to three working days, split it by user-visible outcome while keeping each shipped slice financially consistent.

### 2. Data and edge-case check

Answer briefly:

- Does this affect a ledger fact, balance, allocation, obligation, cycle, forecast, or the distinction between actual and expected money?
- Does it change persisted data, schema, state transitions or correction behavior?
- What must be atomic, and what must remain unchanged if the operation fails?
- Which existing rules and decisions apply?

List the top three to five relevant cases as Given / When / Then. Fewer are fine for a small change when they cover its risks. Include numerical inputs and expected outputs for calculations. Resolve an open accounting rule before implementing or shipping the affected path; do not invent an answer.

### 3. Build and verify a vertical slice

Break the slice into only the work it needs:

1. **Database:** Add or migrate the minimum data and constraints.
2. **Core engine / API:** Implement behavior and preserve transaction and ledger invariants.
3. **UI state:** Connect the flow and its relevant empty, success, validation and failure states to real behavior.
4. **Tests:** Cover meaningful rules, edge cases and failure behavior; use integration tests when persistence or atomicity matters.

A layer can be marked “not needed.” Aim for a shippable slice in one to three working days. Split larger work into smaller outcomes, but do not expose a financial operation that leaves data inconsistent.

Before shipping, run relevant tests and checks, exercise the flow and verify that failure does not leave partial financial changes. For calculations, check expected values with independent examples, including boundaries and rounding. Record known limitations in the change description.

### 4. Post-ship document sync

Update living references only when the shipped change makes them materially inaccurate:

- `ARCHITECTURE.md`: changed schema, critical state transition or integrity invariant.
- `FINANCIAL_RULES.md`: changed formula, accounting rule, definition or calculation boundary.
- `DECISIONS.md`: a consequential product, accounting or architecture decision; record what and why.
- `V1_SPEC.md`: an explicitly approved change to a core v1 capability, exclusion or user flow.
- Relevant prototype: a materially changed represented UI state.
- `TEST_SCENARIOS.md`: a case needed to implement or understand behavior; executable tests are the lasting verification.

Do not rewrite unrelated documentation or create a status report for every slice. If nothing substantive changed, no documentation update is required.

## Guardrails

- **Size limits:** Keep `README.md` under 100 lines, `V1_SPEC.md` under 200 lines, `ARCHITECTURE.md` under 400 lines and this workflow under 150 lines. Keep a feature brief to one page. Never omit financial correctness to meet a cap.
- **Rules and scenarios:** Keep each concise. Do not discard a distinct formula, rounding rule, numerical example or important edge case to satisfy a line cap. Remove duplication as tests take over.
- **Decision log:** Keep entries short: date when known, decision and reason. Mark unresolved questions `OPEN`; do not log routine coding choices.
- **No speculative infrastructure:** Before v1 launch, introduce an abstraction, service, event framework or integration only when a current slice needs it. Use a modular monolith and simple in-process boundaries.
- **Prefer tests to checklists:** Put deterministic behavior and failure cases in executable tests. Do not maintain a parallel traceability matrix, acceptance catalog or ceremonial readiness gate.
- **Protect financial truth:** Keep actual and forecast state distinct; follow agreed money, rounding, transaction and correction rules. Do not trade integrity for a smaller slice.
- **Use AI with bounded context:** Provide the feature brief, relevant active rules, affected code and prototype. Ask for uncertainties, a small implementation and appropriate checks. Review financial assumptions and the diff yourself.
- **Archive, do not recreate:** Consult historical documents as needed, but do not restore their process ceremony without a deliberate decision.
