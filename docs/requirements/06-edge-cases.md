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
