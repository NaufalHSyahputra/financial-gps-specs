# V1 Financial Edge Cases

**Document ID:** EC-V1  
**Status:** APPROVED / FROZEN — Normative

Types: **REJECT**, **WARN/CONFIRM**, **ACCEPT + SURFACE**, **DERIVE**.

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-MONEY-001` | Fraction unsupported by currency minor unit | Reject invalid canonical amount; no floating-point storage. | REJECT |
| `EC-CYCLE-001` | Anchor 31 in shorter month | Nominal anchor = last calendar day, then apply adjustment policy. | DERIVE |
| `EC-CYCLE-002` | Nominal anchor Saturday, PREVIOUS | Walk backward to previous weekday. | DERIVE |
| `EC-CYCLE-003` | Nominal anchor Sunday, NEXT | Walk forward to next weekday. | DERIVE |
| `EC-CYCLE-004` | Adjusted next-month anchor crosses month boundary | Use resulting effective date; cycle remains previous effective anchor to next effective anchor with no overlap/gap. | DERIVE |
| `EC-CYCLE-005` | Public holiday on weekday | Treat as working day in V1 unless holiday mechanism is explicitly added later. | DERIVE |
| `EC-CYCLE-006` | New cycle starts | Account/liability/allocation state persists; prior Budgets close; new Budget Spent starts only in newly created instances. | DERIVE |
| `EC-CYCLE-007` | Anchor/policy changes during active cycle | Keep the active cycle unchanged. Financial Calendar resolves the transition beginning at its next boundary. | DERIVE |
| `EC-CYCLE-008` | New-anchor transition is under 15 days | Do not create a standalone cycle; merge the transition with the following regular cycle as an Extended Cycle. | DERIVE |
| `EC-CYCLE-009` | New-anchor transition is exactly 15 days or longer | Create a standalone Short Cycle before regular cycles under the new anchor. | DERIVE |
| `EC-TRX-001` | Owned asset transfer | No income/expense/Budget Spent. | DERIVE |
| `EC-TRX-002` | Expense creates Funding Deficit | Keep truthful Expense; surface deficit. | ACCEPT + SURFACE |
| `EC-TRX-003` | Expense causes allocations > Available | Keep Expense; create allocation shortfall; STS unresolved until explicit resolution. | ACCEPT + SURFACE |
| `EC-TRX-004` | Void expense | Reverse balance and Budget Spent effects; recalc STS/Forecast/Insights. | ACCEPT + SURFACE |
| `EC-BUD-001` | Budget Amount=0, Spent=0 | Utilization=0%, no overspend. | DERIVE |
| `EC-BUD-002` | Budget Amount=0, actual expense >0 | `OVERSPENT_UNBUDGETED`; do not display infinite percentage. | DERIVE |
| `EC-BUD-003` | Utilization exactly 80% | `APPROACHING_LIMIT`. | DERIVE |
| `EC-BUD-004` | Utilization exactly 100% | `LIMIT_REACHED`. | DERIVE |
| `EC-BUD-005` | Utilization >100% | `OVERSPENT`, show overspend amount. | DERIVE |
| `EC-BUD-006` | Expense maps to budget | Expense reduces actual money once and increases Budget Spent; budget does not deduct cash again. | DERIVE |
| `EC-BUD-007` | Unused budget at cycle close | Does not create money. Existing unspent Flexible remains; old budget closes. | DERIVE |
| `EC-BUD-008` | Apply previous-cycle template | Create new Budget IDs, same proposed definitions after user confirmation, Spent=0. | ACCEPT |
| `EC-BUD-009` | Refund reverses prior eligible expense | Reduce affected Budget Spent up to attributable reversed spending; do not create duplicate income semantics. | DERIVE |
| `EC-BUD-010` | Expense recategorized | Remove from old mapped budget and add to new mapped budget for same effective cycle. | DERIVE |
| `EC-PACE-001` | Consumption exactly cycle progress+5pp | `ON_PACE`; only `>` tolerance is ahead. | DERIVE |
| `EC-PROJ-001` | Day 1 or Day 2 of cycle | Utilization/progress/pace may be shown; numeric Projected Spending/Overspend is `NOT_YET_ELIGIBLE`. | DERIVE |
| `EC-PROJ-001A` | Day 3 with Budget Spent=0 | Projection becomes eligible; Projected Spending=0 and Projected Overspend=0. | DERIVE |
| `EC-PROJ-002` | Transaction history known incomplete | Projection=`INSUFFICIENT_DATA`; no numeric invented projection. | DERIVE |
| `EC-PROJ-003` | Already overspent | Daily amount to stay within original budget=0; explain overspend cannot be undone. | DERIVE |
| `EC-INC-001` | Expected salary before obligation | Salary contributes zero to current STS; may improve Forecast only. | DERIVE |
| `EC-INC-002` | Uncertain freelance income | Base forecast excludes it; scenario may include it and must label assumption. | DERIVE |
| `EC-INC-003` | Actual income arrives early/late/with a different amount and user explicitly links it | Posted actual Income is authoritative; mark the Expected Income `RECEIVED` without duplicate inflow. | ACCEPT + SURFACE |
| `EC-INC-004` | Expected income missed/cancelled | Remove future forecast event; current actual STS was never dependent on it. | DERIVE |
| `EC-INC-005` | Similar actual Income exists but user does not link it | Keep actual Income valid and unlinked; do not mark Expected Income `RECEIVED` from amount/date/description/source similarity. | DERIVE |
| `EC-INC-006` | Late actual receipt is explicitly linked to a `MISSED` occurrence | Transition `MISSED` to `RECEIVED`; actual amount/time is authoritative. | ACCEPT + SURFACE |
| `EC-STS-001` | Raw STS<0 | Display STS=0 and Funding Deficit=abs(raw). | DERIVE |
| `EC-STS-002` | Flexible=0, Unallocated>0 | STS=0; Unallocated is not automatic spend permission. | DERIVE |
| `EC-STS-003` | Locked/Reserved could cover deficit | Do not automatically release. | ACCEPT + SURFACE |
| `EC-STS-004` | Allocation shortfall | Authoritative STS/Daily STS unresolved. | ACCEPT + SURFACE |
| `EC-DSTS-001` | Final cycle day | Remaining days=1. | DERIVE |
| `EC-OBL-001` | Overdue unpaid obligation | Remains STS-applicable and visible. | ACCEPT + SURFACE |
| `EC-OBL-002` | Partial payment | Remaining due stays active. | DERIVE |
| `EC-OBL-003` | Reservation fully covers obligation | Flexible STS deduction for that obligation=0. | DERIVE |
| `EC-OBL-004` | Reservation partially covers applicable obligation | Deduct only `max(0, Applicable Obligation Amount - Applicable Reserved Coverage)` from Flexible. | DERIVE |
| `EC-REC-001` | Staged recurring periods overlap | Reject rule/stage edit. | REJECT |
| `EC-REC-002` | Stage boundary date | Occurrence uses stage whose effective interval contains occurrence date; intervals must be unambiguous. | DERIVE |
| `EC-CC-001` | Card purchase | Expense recognized at purchase; liability increases. | DERIVE |
| `EC-CC-002` | Card payment | Asset/liability decrease; no second category expense. | DERIVE |
| `EC-CC-003` | Purchase moved across cutoff by correction | Reassign statement/obligation atomically; purchase expense remains once. | ACCEPT + SURFACE |
| `EC-INST-001` | Installment includes fees/interest | Use explicit actual installment schedule, not principal/count assumption. | DERIVE |
| `EC-INST-002` | Conversion would duplicate original repayment obligation | Supersede original future repayment representation atomically. | ACCEPT + SURFACE |
| `EC-BNPL-001` | Rp800k purchase + Rp25k fee | Recognize purchase expense and explicit fee expense/liability per timing rule; total liability Rp825k when both incurred; repayment is not duplicate purchase expense. | DERIVE |
| `EC-ADV-001` | Advisor wants numeric corrective amount | Must use engine output; omit amount if required input/projection unavailable. | DERIVE |
| `EC-REF-001` | Delete record with dependent settlement/reconciliation | Reject destructive deletion; use void/cancel/supersede. | REJECT |
| `EC-CONC-001` | Concurrent allocation edits oversubscribe | Serialize/reject one commit; never silently violate normal conservation. | REJECT |

| `EC-ALLOC-NEW-001` | Starting balance created during onboarding | Newly available amount is entirely Unallocated unless explicitly allocated; STS does not increase from Unallocated. | DERIVE |
| `EC-ALLOC-NEW-002` | Salary received | Actual balance/Available increase; new delta remains Unallocated until explicit allocation. | DERIVE |
| `EC-ALLOC-NEW-003` | Freelance income received | Actual balance/Available increase; new delta remains Unallocated until explicit allocation. | DERIVE |
| `EC-ALLOC-NEW-004` | User partially allocates newly available money | Allocated portion moves to selected role; remainder stays Unallocated. | ACCEPT |
| `EC-ALLOC-NEW-005` | User reallocates Flexible/Reserved/Locked | Apply explicit atomic allocation change; conservation must hold and no implicit destination is inferred. | ACCEPT |
| `EC-BUD-CAP-001` | Total Active Budget < Flexible | Valid; no budget-capacity warning. | DERIVE |
| `EC-BUD-CAP-002` | Total Active Budget = Flexible | Valid; no budget-capacity warning. | DERIVE |
| `EC-BUD-CAP-003` | Total Active Budget > Flexible | Valid; do not block; show `BUDGET_EXCEEDS_FLEXIBLE` by exact difference. No money is reserved. | WARN/CONFIRM |
| `EC-BUD-CAP-004` | Budget created/edited/deleted | Flexible/Reserved/Locked/Unallocated/Available do not change solely because of Budget mutation. | DERIVE |
| `EC-STS-BND-001` | Obligation due final day of active cycle | It is STS-applicable. | DERIVE |
| `EC-STS-BND-002` | Obligation due one day after active cycle | It does not reduce current STS; Planning/Forecast evaluates future risk. | DERIVE |
| `EC-FDEP-001` | Post-cycle obligation due before supporting expected income | No Funding Dependency to that later income; surface projected deficit/risk if actual-only funds are insufficient. | DERIVE |
| `EC-FDEP-002` | Post-cycle obligation due after EXPECTED_STABLE income and actual-only funds insufficient | Funding Dependency exists; current STS remains unchanged. | DERIVE |
| `EC-FDEP-003` | Supporting EXPECTED_STABLE income becomes MISSED or moves after obligation | Remove dependency-to-income assumption and surface projected funding deficit/risk. | DERIVE |
| `EC-FDEP-004` | Supporting expected income never arrives | It never becomes Actual; after expected time it is MISSED and forecast/funding risk recalculates. | DERIVE |
| `EC-FDEP-005` | Sufficient actual funding becomes available before obligation | Funding Dependency disappears after recalculation. | DERIVE |
| `EC-FCST-BASE-001` | EXPECTED_STABLE future income | Included in Base Forecast, never in current STS. | DERIVE |
| `EC-FCST-BASE-002` | EXPECTED_UNCERTAIN future income | Excluded from Base Forecast. | DERIVE |
| `EC-FCST-SCN-001` | User selects uncertain income scenario | Add selected uncertain event to Scenario Forecast only; Actual and Base remain unchanged. | DERIVE |
| `EC-FCST-RECV-001` | Expected Income explicitly linked to received actual Income | Mark Expected Income `RECEIVED`; actual amount/time authoritative; newly available delta remains Unallocated. | ACCEPT + SURFACE |
| `EC-FCST-LATE-001` | Expected time passes without receipt | Expected Income becomes MISSED; never silently becomes actual. | DERIVE |
| `EC-PROJ-004` | Budget created after cycle Day 3 | Projection is eligible immediately using cycle ElapsedCycleDays and all eligible mapped cycle spending. | DERIVE |
| `EC-PROJ-005` | Backdated eligible expense | Recompute Budget Spent, utilization, pace, projection/overspend if eligible, and Insights. | ACCEPT + SURFACE |
| `EC-PROJ-006` | Deleted/voided eligible expense | Remove its budget contribution and recompute projection/insights. | ACCEPT + SURFACE |

## Canonical Numerical Scenarios

### `EC-NUM-001` Expected income never bridges STS

```text
Flexible actual                 Rp5,000,000
Expected salary                Rp10,000,000
Current-cycle obligation        Rp6,000,000
Raw STS                        -Rp1,000,000
Displayed STS                   Rp0
Funding Deficit                 Rp1,000,000
```

Forecast may show improved future position after salary, but current STS remains zero.

### `EC-NUM-002` Budget projection

```text
Cycle days                      30
Elapsed days                    18
Budget                          Rp1,500,000
Spent                           Rp1,280,000
Average/day                     Rp71,111.11...
Projected (round down)          Rp2,133,333
Projected overspend             Rp633,333
Future days                     12
Remaining budget                Rp220,000
Max avg/day to stay in budget   Rp18,333
```

### `EC-NUM-003` No financial rollover

```text
Old budget                      Rp1,500,000
Old spent                       Rp1,200,000
Unused budget                   Rp300,000
```

At cycle close, no Rp300,000 transaction is created. If the unspent cash was Flexible, it remains Flexible. A template can create a new Rp1,500,000 budget with Spent=0.

Every `EC-*` MUST map to automated domain/acceptance coverage where applicable.

## Ledger clarification edge cases — CR-LDG-001

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-LDG-001` | User attempts ordinary new entry against archived account | Require restoration first; history remains correctable and eligible archived money still counts. | REJECT / GUIDE |
| `EC-LDG-002` | Expense exceeds tracked cash balance | Save truthful expense; show negative tracked balance; account contributes zero to Available while negative; surface allocation shortfall if present. | ACCEPT + SURFACE |
| `EC-LDG-003` | Payment exceeds recorded liability outstanding | Reject without partial mutation and suggest checking missing or incorrect transactions. | REJECT |
| `EC-LDG-004` | Partial linked refund | Reverse attributable eligible spending only; no duplicate ordinary income. | DERIVE |
| `EC-LDG-005` | Transfer of Rp100,000 with Rp2,500 fee | Transfer principal is not spending; separately recognized fee expense is Rp2,500. | DERIVE |
| `EC-LDG-006` | Actual transaction predates Calendar historical floor | Do not invent a cycle; mark affected cycle-dependent budget results unavailable. Mark affected cycle-based historical reports incomplete or unavailable. | ACCEPT + SURFACE |
| `EC-LDG-007` | Rename/archive category used by historical transaction | Preserve reference; show current name and retain rename audit; hide archived category from ordinary new entry. | ACCEPT + SURFACE |
| `EC-LDG-008` | User corrects or voids linked income | Show linked occurrence; obtain appropriate confirmation; preserve audit and update dependent records atomically. On unlink, use EXPECTED before expected time or MISSED after it; if intervening changes make the state unsafe to determine, require guided resolution without partial save. | ACCEPT + SURFACE |
| `EC-LDG-009` | Unlinked refund later linked to original purchase | Before linking, increase receiving account and report separately without automatic budget change. After user confirms the link, recalculate attributable eligible spending with audit; no duplicate inflow. | ACCEPT + SURFACE |
| `EC-LDG-010` | Transfer with fee is voided but fee really was charged | Suggest voiding both; permit keeping the fee after user review, preview effects and atomically commit the selected changes. | ACCEPT + SURFACE |
| `EC-LDG-011` | Linked income removed after an intervening change makes expected state unsafe to determine | Show conflict and require guided resolution; commit no partial correction. | REJECT / GUIDE |
| `EC-LDG-012` | Purchase Rp500,000 already has Rp300,000 linked refunds; attempt to link another Rp250,000 | Reject the link, explain Rp200,000 remains; no partial mutation. | REJECT |
| `EC-LDG-013` | Edit a purchase to Rp200,000 when Rp300,000 of valid refunds are linked | Show conflict; require explicit resolution; do not partially commit or silently alter refunds. | REJECT / GUIDE |
| `EC-LDG-014` | Refund to fully paid card would produce positive card credit | Explain unsupported representation; do not drop the credit or record an unreceived bank inflow. | SURFACE |
| `EC-LDG-015` | Asset opening snapshot Rp1,000,000 on Nov 1; add Rp100,000 expense on Oct 25 | Retain the expense in history; do not subtract it from the Nov 1 snapshot or subsequent balance again. Historical budget eligibility remains governed by Calendar and Budget. | ACCEPT + SURFACE |
| `EC-LDG-016` | Date-only transaction and asset opening snapshot have same date and no ordering times | Ask user whether before or after snapshot; preserve choice; include effect in subsequent balance only if after. | ACCEPT + SURFACE |
| `EC-LDG-017` | Card outstanding Rp60,000; card refund Rp100,000 | Explain full refund cannot be represented as card credit; do not commit only Rp60,000 or discard Rp40,000; do not record unreceived bank refund. | REJECT / GUIDE |
| `EC-LDG-018` | Correct an expense from Nov 2 to Oct 31, crossing a Nov 1 asset opening snapshot | Preview the impact; retain history and eligible budget effects but do not subtract the corrected pre-snapshot expense from the snapshot or subsequent balance. | ACCEPT + SURFACE |
| `EC-LDG-019` | Correct a Rp500,000 credit purchase to Rp300,000 when a Rp500,000 payment exists | Show payment conflict; require explicit resolution before saving; preserve prior facts until a valid set of changes commits atomically. | REJECT / GUIDE |
| `EC-LDG-020` | Edit an asset opening snapshot's amount or date | Preview current-balance impact and transactions whose relationship changes; retain prior snapshot in audit; ask before/after when same-day order is unclear; recalculate after confirmation. | ACCEPT + SURFACE |
| `EC-LDG-021` | Credit-card debt Rp300,000 on Nov 5; Rp200,000 purchase Nov 10; enter Rp400,000 payment dated Nov 5 | Validate debt at payment time and affected later history; show conflict and require correction; no partial mutation or silently inferred ordering. | REJECT / GUIDE |
| `EC-LDG-022` | Same-day date-only card purchase and payment can be valid in one order and invalid in another | Ask which occurred first, preserve choice, and validate against that order; no automatic guess or partial write. | ACCEPT + SURFACE |
| `EC-LDG-023` | Recorded card debt immediately before an old payment is unknown | Show unknown history and guide user to provide or correct needed starting information before validation/save. Do not assume sufficient debt. | REJECT / GUIDE |
| `EC-LDG-024` | Historical card debt is unknown and user provides an earlier known balance | Identify affected later transactions, preview impact and preserve prior information in audit; do not count debt twice. Compare against the later opening balance without adding the two; flag mismatch or indeterminate history. | ACCEPT + SURFACE |
| `EC-LDG-025` | Editing a card transaction invalidates a previous same-day ordering choice | Revalidate affected history; retain the choice only if applicable, ask again if needed for payment validity, and prevent partial save on conflict. | ACCEPT + SURFACE |
| `EC-LDG-026` | Earlier card debt Rp700,000 plus recorded intervening effects yields Rp950,000 at a later opening of Rp1,000,000 | Show Rp50,000 discrepancy; retain both recorded amounts, do not invent a transaction, and mark affected historical calculation incomplete/unavailable until resolved. | ACCEPT + SURFACE |
| `EC-LDG-027` | Earlier card checkpoint and later opening lack sufficient intervening history or ordering for a reliable comparison | Show comparison as indeterminate; do not invent an exact difference or assume sufficient debt to validate an old payment. | ACCEPT + SURFACE |
| `EC-LDG-028` | Card purchase and payment have the same explicit timestamp and order changes payment validity | Ask for and preserve the user's order; if unknown, return indeterminate and do not partially commit the operation requiring a definite result. | ACCEPT + SURFACE |
| `EC-LDG-029` | Transaction and opening snapshot or historical checkpoint share an exact timestamp and the result depends on their order | Ask whether transaction is before or after the balance, preserve choice; if unknown return indeterminate. | ACCEPT + SURFACE |
| `EC-LDG-030` | Equal-time events produce the same result in every applicable order | Do not ask for ordering solely for the tie; retain deterministic financial result. | DERIVE |
| `EC-LDG-031` | User selects the same account as source and destination for a transfer | Reject without partial mutation and explain that source and destination must differ. | REJECT |
