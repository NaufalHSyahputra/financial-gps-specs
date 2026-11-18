# V1 User Flows

**Document ID:** UF-V1  
**Status:** APPROVED / FROZEN — Normative

## `UF-ONB-001` Short onboarding
1. Choose currency/region if required and Financial Timezone.
2. Enter primary payday/cycle anchor.
3. Choose `NONE|PREVIOUS_WORKING_DAY|NEXT_WORKING_DAY`.
4. Enter starting Cash balance; system may create default Cash account. The starting balance is Unallocated by default.
5. Enter app. Home/Accounts show Available and Unallocated even when STS is zero. Additional accounts are optional later.

## `UF-TRX-001` Quick expense
`+ → calculator amount → Category → Account → Save`.
After save: actual account balance, Budget Spent, allocation consistency, STS, Forecast starting state and Insights recalculate. Optional metadata never blocks ordinary path.

## `UF-TRX-002` Transfer
Choose source → a different destination → amount → save atomically. Account balances change; transfer principal is not spending/Budget Spent. Reject a same-account transfer without partial mutation (`FR-LDG-018`). An optional fee is a separately recognized expense under `FR-LDG-008`.

## `UF-INC-001` Expected income planning
Enter amount/date/source and `stable|uncertain`. Planning/Forecast updates. Current Available/Flexible/STS do not change.

## `UF-INC-002` Receive income and allocate
1. Record an actual Income transaction in Financial Accounts & Ledger.
2. Optionally select a relevant Expected Income occurrence; no selection is valid and no fuzzy/heuristic suggestion may execute as a link.
3. If selected, Income Management changes `EXPECTED` or `MISSED` to `RECEIVED`; otherwise Expected Income remains unchanged.
4. Actual account/Available increases and the newly available delta remains Unallocated.
5. System asks/suggests: Flexible, Reserved purpose, Locked purpose, or leave Unallocated.
6. Suggestions do not execute automatically; only the user’s explicit choice changes allocation.

## `UF-CYCLE-001` Change payday anchor or adjustment policy
User changes cycle configuration → active cycle remains unchanged → Financial Calendar derives the transition from the next boundary. If the transition is under 15 days, merge it with the following regular cycle as an Extended Cycle. If it is at least 15 days, create a standalone Short Cycle, then continue regular cycles under the new configuration.

## `UF-BUD-001` Create budget
Choose current cycle → purpose/category mapping → amount → save. Spent derives from actual mapped expenses in cycle. Saving the Budget moves no money. If aggregate active Budget exceeds Flexible, save succeeds and the system shows the exact non-blocking `BUDGET_EXCEEDS_FLEXIBLE` warning.

## `UF-BUD-002` Budget warning and pace
Open Budget → see utilization + cycle progress. At 80% show approaching limit; at 100% reached; above 100% overspent. On Cycle Day 1–2 do not show projected spend/overspend. From Day 3 onward, if projection inputs are complete, show projected spend/overspend and deterministic max average daily spend to stay within original budget.

## `UF-BUD-003` New cycle/template
At new cycle, Budget list starts empty. User selects “Use previous cycle as template” → reviews prior definitions/amounts → Adjust or Apply → system creates new Budget instances with Spent=0. No cash movement occurs.

## `UF-STS-001` Understand STS
Open Home → STS → “Why this amount?”. Explanation shows Flexible, current-cycle applicable obligations, reservation coverage, Funding Deficit if any, cycle horizon, and explicitly excludes Expected Income until received.

## `UF-FCST-001` Compare forecast assumptions
Planning shows actual starting state and future events. Base Forecast always includes `EXPECTED_STABLE` income and excludes `EXPECTED_UNCERTAIN`. User may view a Scenario Forecast that adds selected uncertain income. Neither forecast alters Home STS.

## `UF-FCST-002` Boundary Funding Dependency
A future post-cycle obligation is not deducted from current STS. Planning evaluates the actual-only projected path before the obligation. If actual-only funds are insufficient and `EXPECTED_STABLE` income is scheduled before the obligation, show Funding Dependency with the obligation and supporting unreceived income. If that income becomes MISSED/delayed beyond the obligation, replace the dependency with projected funding risk/deficit. If sufficient actual funding arrives, the dependency disappears after recalculation.

## `UF-OBL-001` Recurring/staged KPR
Create recurrence + due timing + effective amount stages. System validates no overlap and generates occurrences with the stage valid on each occurrence date.

## `UF-CC-001` Credit-card lifecycle
Purchase → recognize Expense + liability → assign statement → create repayment obligation → pay from asset → reduce asset/liability + settle obligation. Payment creates no second category expense.

## `UF-BNPL-001` PayLater lifecycle
Purchase → recognize purchase Expense + liability and explicit fees when incurred → create single or installment obligations → payment settles liability/obligation without duplicate purchase expense.

## `UF-ALLOC-001` Resolve allocation shortfall
Truthful expense causes allocations > Available → retain expense → show shortfall → mark STS unresolved → user explicitly releases/reallocates → conservation restored → STS resumes.

## `UF-ADV-001` Contextual deterministic insight
Budget/Home/Planning detects rule condition → show What Happened → Why It Matters → Forecast/likely deterministic result → Impact → Possible Action. Any amount is read from engine outputs; if unavailable, omit numeric recommendation.

## `UF-EDIT-001` Correct history
Edit/void transaction → validate dependencies → commit correction atomically → propagate balances, budgets, STS, Forecast, liabilities/settlements, and Insights.

## Ledger clarification flows — CR-LDG-001

### `UF-LDG-001` Archived account
Choose archived account → restore it before ordinary new transaction entry. Historical correction remains accessible; balance and history are preserved.

### `UF-LDG-002` Correct tracked balance
Open account → choose balance adjustment → enter actual discrepancy and explanation → review → save auditable adjustment. Show separately from ordinary income/expense and recalculate actual position and allocation consistency. Show the adjustment separately in reports; forecasts start from the corrected actual balance.

### `UF-LDG-003` Transfer with fee
Choose source, destination and principal → optionally enter fee → review → save. Show principal as transfer and fee as separate expense. During correction, show both records, preview effects and allow explicit choices for each. On transfer void, suggest voiding both while allowing a fee actually charged to remain; save the chosen changes together.

### `UF-LDG-004` Refund
Choose original purchase and record a full or partial refund by default. An unlinked refund is labeled and reported separately, with no automatic category budget change. If the user later selects the original purchase, confirm the link and recalculate the attributable budget effect.

### `UF-LDG-005` Correct linked income
Open actual income → choose edit or void → display the linked expected occurrence and required impact → obtain confirmation where a correction choice is needed → commit applicable dependent updates atomically. After unlinking, the occurrence is EXPECTED if its expected time has not passed, otherwise MISSED. If intervening changes make the state unsafe to determine, show the conflict and require guided resolution without a partial save.

### `UF-LDG-006` Pre-calendar transaction
Enter an otherwise valid transaction before the Calendar historical floor → save actual fact → show cycle-dependent budget results as unavailable rather than inventing a cycle. Show its account effect and mark affected cycle-based historical reports incomplete or unavailable.

### `UF-LDG-007` Correct wrong account class
Create the correct account → inspect affected transactions and preview impact → explicitly correct each applicable transaction through the guided process → preserve audit → archive the old account when appropriate. Never silently reinterpret history.

### `UF-LDG-008` Refund validation and conflict
When linking a refund, display remaining refundable purchase amount; reject a link exceeding it. When a purchase correction or void would invalidate existing links, show the affected refunds and require explicit resolution before saving.

### `UF-LDG-009` Historical transaction and opening snapshot
When entering an asset transaction before the account's opening snapshot, show that it remains in history and does not change the snapshot or later balance. Cycle-based reporting follows calendar availability and is marked incomplete or unavailable where applicable.

### `UF-LDG-010` Same-day opening snapshot
When a date-only transaction shares an opening-snapshot date and no times establish their order, ask whether it happened before or after the snapshot. Preserve the choice and explain that only an after-snapshot transaction changes the subsequent balance.

### `UF-LDG-011` Unsupported card credit from refund
When a card refund would exceed outstanding, explain that V1 cannot represent the full card credit. Do not record only part of the refund or an unreceived bank refund; guide the user to correct erroneous history or record a bank refund only once actually received.

### `UF-LDG-012` Correct transaction across opening snapshot
Edit transaction date → preview effect on opening-snapshot relationship, balance and eligible budget → supply before/after choice if ordering is unclear → confirm → preserve audit and recalculate applicable results.

### `UF-LDG-013` Correct purchase with dependent payment
Edit a credit purchase → identify whether the change would invalidate a recorded payment → show the conflict and require user-guided resolution → commit a valid set of changes together, or keep existing records unchanged.

### `UF-LDG-014` Edit opening snapshot
Edit opening amount or date → preview current-balance impact and transactions whose snapshot relationship changes → specify before/after where same-day order is unclear → confirm → retain the previous snapshot in audit and recalculate affected results.

### `UF-LDG-015` Backdated credit-card payment
Enter payment with financial date → evaluate recorded debt at payment time and affected later history → show any unsupported card-credit conflict and require correction. If same-day date-only transaction ordering affects validity, ask which happened first and preserve the choice. If debt immediately before the old payment is unknown, explain and guide the user to provide or correct necessary starting information before validation/save. The detailed starting-information workflow remains to be designed.

### `UF-LDG-016` Provide earlier known card debt
Historical payment cannot be validated → explain missing debt → user provides an earlier known amount and effective date → identify later transactions to be checked → preview impact → confirm and retain audit. Use the earlier balance as a historical checkpoint and compare intervening activity with the later opening balance without adding the balances. On mismatch, show the difference and guide investigation without changing either balance or creating a transaction; mark affected historical calculations incomplete or unavailable. If comparison inputs are insufficient, show indeterminate rather than inventing a difference.

### `UF-LDG-017` Revalidate card transaction order
User edits an affected card transaction → revalidate history → retain an ordering choice only if it still applies → ask for a new choice if needed for payment validity → show conflicts → save a valid set together or preserve existing records.

### `UF-LDG-018` Resolve equal-time order
When equal-time events or a transaction and balance checkpoint would yield different results depending on their order → explain the impact → ask the user to choose the real ordering → preserve the choice and validate. If the user cannot determine the order, show an indeterminate result and do not commit an operation requiring a definite result. Ask again if a later edit invalidates the earlier choice. Do not ask solely when order cannot change the result.
