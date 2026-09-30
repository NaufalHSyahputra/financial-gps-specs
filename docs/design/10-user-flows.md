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
Choose source → destination → amount → save atomically. Account balances change; spending/Budget Spent do not.

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
