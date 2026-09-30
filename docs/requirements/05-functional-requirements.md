# V1 Functional Requirements

**Document ID:** FR-V1  
**Status:** APPROVED / FROZEN — Normative

## 1. Onboarding & Settings

### `FR-ONB-001` Short onboarding
Collect calculation currency/region as needed, Financial Timezone, cycle/payday anchor, weekday adjustment policy, and starting Cash balance. A default Cash asset account MAY be created. Other accounts MUST NOT be required to finish onboarding. Starting Cash MUST remain Unallocated unless the user explicitly allocates it during or after onboarding.

### `FR-CYCLE-001` Configure cycle
User can set anchor day and `NONE|PREVIOUS_WORKING_DAY|NEXT_WORKING_DAY`. System previews effective next boundary and recalculates cycle-dependent outputs after changes.

## 2. Accounts

### `FR-ACC-001` Asset accounts
Create/edit/archive generic cash, bank, e-wallet, savings or other supported assets with liquidity eligibility.

### `FR-ACC-002` Liability accounts
Create/edit/archive generic credit card, PayLater or other liabilities. Credit limit is informational and never Available Money.

### `FR-ACC-003` Archive/history
Archiving preserves economic history and non-zero balance/liability effects.

## 3. Transaction Capture

### `FR-TRX-001` Quick expense
Default path MUST support calculator-style amount input → Category → Account → Save. Amount, category, and account are primary fields. Merchant/notes/tags/custom time/attachments are secondary unless a specific rule requires them.

### `FR-TRX-002` Income
Posted Income increases actual receiving asset balance and, by default, increases Unallocated by the same newly available amount while existing explicit allocations remain unchanged. The system MUST then allow explicit destination allocation to Flexible/Reserved/Locked or leave it Unallocated; it MUST NOT hardcode or silently execute allocation by income source. Previous choices may be suggestions only.

### `FR-TRX-003` Expense
Posted asset-funded Expense reduces actual account balance, updates eligible Budget Spent, allocation consistency, STS, Forecast starting state, and Insights.

### `FR-TRX-004` Transfer
Owned-account Transfer uses atomic source/destination legs and does not count as income, expense, or Budget Spent.

### `FR-TRX-005` Edit/void/correction
Correction MUST propagate to balances, liabilities, budgets, STS, Forecast, obligations/settlements, and Insights as applicable. Audit history is preserved.

## 4. Money Allocation

### `FR-ALLOC-001`
Show Flexible, Reserved, Locked, Unallocated and Available with conservation relationship.

### `FR-ALLOC-002`
User explicitly changes allocation roles. Normal allocation commands cannot oversubscribe Available.

### `FR-ALLOC-003`
If a truthful realized event causes allocation shortfall, retain the event, show exact shortfall, mark STS unresolved, and require explicit allocation resolution. Never silently choose which intent to reduce.

### `FR-ALLOC-004`
Released Reserved/Locked defaults to Unallocated unless user chooses another destination.

## 5. Budget

### `FR-BUD-001` Create cycle budget
User creates a Budget with amount, name/purpose, Financial Cycle, and category mapping(s). Budget and Allocation remain separate. Budget creation MUST NOT move/reserve money. If Total Active Budget exceeds current Flexible Money, save remains allowed and the system MUST show `BUDGET_EXCEEDS_FLEXIBLE` with the difference.

### `FR-BUD-002` Budget health
For every Budget show Budget Amount, Spent, Remaining or Overspend, utilization, status, and cycle days/progress.

### `FR-BUD-003` Warning thresholds
Use default deterministic states: `<80% ON_TRACK_LIMIT`, `80..<100% APPROACHING_LIMIT`, `100% LIMIT_REACHED`, `>100% OVERSPENT`.

### `FR-BUD-004` Spending pace
Show cycle progress versus budget consumption and `ON_PACE|AHEAD_OF_PLAN` per Financial Engine tolerance.

### `FR-BUD-005` Projection
Budget utilization, cycle progress, and pace may be shown from Day 1. Projected Budget Spending and Projected Overspend MUST NOT be displayed until at least 3 Financial Cycle days have elapsed. Before then show projection unavailable/not yet eligible. From Day 3 onward, use the Financial Engine formula; if history is incomplete, show `INSUFFICIENT_DATA` rather than inventing a number.

### `FR-BUD-006` Cycle close
Old Budget becomes historical. New cycle has no Budget instances until user creates/applies them. Account balances, liabilities, allocations, and persistent obligations do not reset.

### `FR-BUD-007` Previous-cycle template
User can preview previous-cycle budget definitions, adjust them, then Apply to create new-cycle Budget instances with Spent=0.

### `FR-BUD-008` Refund/category correction
Eligible refund or transaction recategorization MUST deterministically recompute Budget Spent for affected budgets/cycles.

## 6. Obligations

### `FR-OBL-001`
Create/edit/cancel/settle manual obligations with amount, due time, description/purpose, source and status.

### `FR-OBL-002`
Partial settlement leaves Remaining Due active; overdue unpaid obligations remain visible and STS-applicable.

### `FR-OBL-003`
Reserved money may link to an obligation and fund it without duplicate STS deduction. `Unfunded Obligation Amount = max(0, Applicable Obligation Amount - Applicable Reserved Coverage)`; STS deducts only this uncovered portion from Flexible.

### `FR-REC-001` Recurring obligations
Create recurring rules for KPR, subscriptions, bills, recurring transfers/fixed payments.

### `FR-REC-002` Staged amounts
Recurring rule supports effective amount periods. Overlap is rejected; correct stage generates each occurrence.

## 7. Expected Income

### `FR-INC-001`
Create Expected Income with amount, expected time, recurrence/source, and simple certainty `EXPECTED_STABLE|EXPECTED_UNCERTAIN`.

### `FR-INC-002`
Expected Income may appear in Planning/Forecast/scenarios but MUST NOT change actual balances, Flexible, STS or Daily STS.

### `FR-INC-003`
Financial Accounts & Ledger owns the posted actual Income; Income Management owns Expected Income lifecycle. While recording actual Income, the user MAY explicitly link a relevant Expected Income occurrence. The link is optional, actual Income may remain unlinked, and an explicit link changes `EXPECTED` to `RECEIVED`. No amount/date/description/source similarity or fuzzy/heuristic mechanism may auto-link or mark it `RECEIVED` in V1.

### `FR-INC-004`
`EXPECTED_STABLE` income MUST be included in Base Forecast. `EXPECTED_UNCERTAIN` income MUST be excluded from Base Forecast and included only in explicit Scenario Forecasts.

### `FR-INC-005` Late/missed expected income
When the expected time passes without a linked actual receipt, the Expected Income becomes `MISSED`; it MUST NOT become Actual Income automatically. Forecast and Funding Dependency recalculate. A later actual receipt MAY change `MISSED` to `RECEIVED` only when the user explicitly links it; actual amount/time is authoritative.

## 8. Safe to Spend

### `FR-STS-001`
Home displays authoritative STS/Daily STS when allocation state is valid, plus Funding Deficit when positive. STS considers only unfunded obligations due within the active Financial Cycle (plus already-overdue obligations); obligations due after the cycle boundary do not reduce current STS.

### `FR-STS-002`
“Why this amount?” shows actual Flexible, applicable obligations, reservation coverage, exclusions (including Expected Income), formula and cycle horizon.

### `FR-STS-003`
Expected Income MUST NEVER be presented as increasing current STS before receipt.

## 9. Forecast / Planning

### `FR-FCST-001`
Planning shows chronological future obligations, recurring events, liability payments, Expected Income and projected positions.

### `FR-FCST-002`
Forecast distinguishes actual starting state from assumptions and base vs uncertain-income scenario where applicable.

### `FR-FCST-003`
Projected Cycle-End Balance must be explainable by included inflows/outflows and assumption set.

### `FR-FCST-004` Funding Dependency
Planning/Advisor MUST detect and explain when a future obligation lacks sufficient actual-only projected liquid funding before its due time and therefore depends on `EXPECTED_STABLE` income scheduled before it. This condition MUST NOT modify current STS. If supporting Expected Income is missed/delayed beyond the obligation, surface projected funding risk/deficit instead of treating it as funded.

## 10. Credit Card / BNPL / Installments

### `FR-CRD-001`
Credit purchase recognizes Expense and increases Liability Outstanding at purchase time.

### `FR-CRD-002`
Support statement/cutoff and due date; statement creates/updates repayment Obligation.

### `FR-CRD-003`
Card payment reduces asset + liability and settles obligation; it MUST NOT create a second purchase-category expense.

### `FR-INST-001`
Support conversion to installment and explicit installment schedule with actual installment amounts, due dates, fees/interest where known.

### `FR-BNPL-001`
Support single-payment and installment PayLater. Purchase expense occurs at purchase; repayment is cash settlement, not duplicate purchase expense.

## 11. Advisor / Insights

### `FR-ADV-001`
Contextual insight may appear in Home, Budget, Planning; no primary Advisor tab required.

### `FR-ADV-002`
Insight structure SHOULD cover what happened, why it matters, likely deterministic outcome, impact, and possible action.

### `FR-ADV-003`
Every numeric statement/action amount MUST reference a Financial Engine derived value; no UI/AI independent arithmetic or invented estimate.

## 12. Dashboard & Navigation

### `FR-NAV-001`
Primary navigation SHOULD remain approximately `Home | Transactions | Budget | Accounts | Planning` across platforms, adapting chrome without changing ownership of concepts.

### `FR-HOME-001`
Home prioritizes STS/Funding Deficit, Daily STS, budget health/attention, upcoming obligations, and contextual insights.

## 13. Atomicity and Error Behavior

- Multi-record economic actions commit atomically.
- Financially unfavorable states are valid and surfaced.
- Validation failure produces no partial canonical mutation.
- Derived values are not directly editable.
- Analytics failure never blocks financial state mutation.
