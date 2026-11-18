# V1 delivery backlog and readiness

**Status:** Proposed ordering; all slices are **not ready** until reviewed and approved. Owner/reviewer: unassigned. This backlog does not reduce V1 scope or override the [tactical roadmap](../../README.md#tactical-ddd-roadmap). Requirements and case IDs link through [traceability](traceability.md) and [acceptance catalog](acceptance-catalog.md).

| Order | Slice and dependencies | Minimum acceptance / remaining work |
|---|---|---|
| 0 | Governance and validation; no dependency | Assign Q-01 owners; review research plan, index and traceability; choose first users and record findings. No claim of validation before evidence. |
| 1 | Financial Calendar foundation; frozen tactical model | Review platform/time inputs and calendar test matrix; cover AC-08 and boundaries. Technical persistence and query selection remain open. |
| 2 | Accounts & Ledger foundation; depends on 1 for cycle context and cross-context contracts for full behavior | Follow the owner-frozen [ledger baseline and open findings](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#30-review-disposition-and-handoff); resolve Q-04–06 and cross-context contracts before implementing affected paths. Cover AC-01–07, AC-14 and AC-19–44 as applicable, including atomic failures, audit and correction. Avoid shipping isolated mutations that violate cross-context consistency. |
| 3 | Income Management; depends on ledger fact and explicit-link contract | Specify lifecycle and integration; cover AC-03, AC-13 and AC-14, missed/late income and no implicit linking. |
| 4 | Money Allocation; depends on ledger changes and income integration | Specify conservation and explicit user intent; cover AC-01, AC-03 and AC-05, including shortfall resolution and no silent release. |
| 5 | Spending Planning; depends on calendar, ledger and allocation contracts | Specify cycle budgets, projections, refunds and corrections; cover AC-10–12 and threshold/pace boundaries. |
| 6 | Financial Commitments; depends on ledger and calendar | Specify obligations, recurring stages, statement and repayment coordination; cover AC-04, AC-15 and AC-18 plus installments/fees. |
| 7 | Financial Intelligence; depends on all preceding authoritative inputs | Specify actual STS, forecast, dependency and explanation; cover AC-09, AC-13, AC-16 and AC-17 with golden calculations and correction propagation. |
| 8 | Cross-context orchestration and consistency; evolves with slices 2–7 | Complete command/event/query catalogs, transaction/failure contracts and recovery, then audit end-to-end flows and global invariants. Do not defer critical consistency until after a slice ships. |
| 9 | Release readiness; depends on all required slices | Complete UX/accessibility review, security/privacy, measured performance, portability/recovery design and tests; meet every frozen NFR release gate. |

Each slice also requires its corresponding UX row, reviewed traceability, concrete negative cases, non-functional requirements and closed or explicitly non-blocking issues. If implementation starts before later designs are complete, expose only an internally coherent vertical slice satisfying immediate consistency; partial financial behavior is not a substitute for the frozen requirements.
