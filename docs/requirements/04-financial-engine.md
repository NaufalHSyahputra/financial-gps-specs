# V1 Financial Engine Specification

**Document ID:** FE-V1  
**Status:** APPROVED / FROZEN — Normative  
**Role:** Canonical source for derived financial values and financial calculation behavior.

## 1. Purpose and Engine Separation

V1 has two strictly separated calculation contexts:

```text
ACTUAL / SAFETY ENGINE                    FORECAST ENGINE
realized balances                         starts from actual state
+ realized transactions                   + expected future income
+ explicit allocations                    - future obligations/outflows
+ applicable obligations                  ± deterministic future events
= current financial guidance              = projected future position
```

`RULE-STATE-001`: A future Expected Income MUST NOT increase Account Balance, Available Money, Flexible Money, Budget Remaining, Raw STS, Displayed STS, or Daily STS before an actual Income transaction is explicitly linked and the occurrence becomes `RECEIVED`.

`RULE-STATE-002`: Forecast outputs MUST identify assumptions separately from actual facts.

## 2. Monetary Arithmetic

### `RULE-MONEY-001` Integer minor units
All canonical money is stored/calculated as integer minor units. Floating point MUST NOT be used.

### `RULE-MONEY-002` Domain amounts
Canonical entered amounts are non-negative; direction comes from transaction/event semantics.

### `RULE-MONEY-003` Division and rounding
Unless a rule states otherwise, division-derived spend guidance is rounded **down toward zero** to the currency minor unit. Percentages MAY retain implementation precision but displayed percentages MUST use a documented consistent rounding method.

## 3. Time Model

### `RULE-TIME-001` Financial timezone
All cycle boundaries, date-only events, “today”, and elapsed/remaining-day calculations use the configured Financial Timezone.

### `RULE-TIME-002` Date-only semantics
A date-only posted transaction belongs to that local financial date. A date-only obligation is due by the end of that local date. Expected Income timing affects Forecast only.

### `RULE-TIME-003` Explicit timestamp
When a timestamp is supplied, use its resolved instant in the Financial Timezone.

## 4. Account Balance and Liability Outstanding

### `RULE-BAL-001` Asset balance
For asset account `a`:

```text
AssetBalance(a,t) = OpeningBalance(a) + Σ posted realized transaction legs affecting a through t
```

Voided legs have zero current effect.

### `RULE-BAL-002` Liability outstanding
For liability account `l`:

```text
LiabilityOutstanding(l,t)
= max(0,
    OpeningLiability(l)
  + recognized purchases/principal
  + recognized fees/interest
  - posted liability payments
  - liability-reducing refunds/credits)
```

Unused credit limit is excluded.

**Negative/zero:** floor at zero unless a separate positive asset/credit-balance representation is explicitly supported.  
**Recalculate:** purchase, fee/interest, refund, payment, void/edit, opening adjustment.

## 5. Available Money

### `RULE-AVAILABLE-001` Definition
Available Money is actual owned liquid money eligible for current allocation.

```text
AvailableMoney(t)
= Σ max(0, AssetBalance(a,t))
  for active or archived asset accounts where liquidity_eligible = true
```

**Exclusions:** liability credit limits; expected income; locked/non-liquidity assets; projected values.  
**Rounding:** none beyond canonical minor units.  
**Zero:** may be zero.  
**Recalculate:** any realized asset balance or liquidity-eligibility change.

## 6. Money Allocation

### `RULE-ALLOC-101` Explicit roles

```text
TotalAllocated = FlexibleMoney + ReservedMoney + LockedMoney
UnallocatedMoney = AvailableMoney - TotalAllocated
```

Normal user allocation mutations MUST satisfy `0 <= TotalAllocated <= AvailableMoney`. `UNALLOCATED` is derived; it is not an allocation record or implicit spend permission.

### `RULE-ALLOC-101A` Newly available money
Any increase in actual Available Money for which no explicit allocation mutation occurs—including onboarding starting balance, received salary, received freelance income, refunds that create newly available liquidity, or other actual inflows—MUST leave existing Flexible/Reserved/Locked amounts unchanged. Therefore the increase appears in `UnallocatedMoney`. No system action may silently convert that delta to Flexible.

**Example:** starting Cash `Rp5,000,000`, no allocations → Flexible `Rp0`, Reserved `Rp0`, Locked `Rp0`, Unallocated `Rp5,000,000`, STS `Rp0`.

### `RULE-ALLOC-102` Allocation shortfall after reality changes
A truthful realized transaction MAY cause `TotalAllocated > AvailableMoney`. The transaction MUST remain recorded. The system enters `ALLOCATION_SHORTFALL`:

```text
AllocationShortfall = max(0, TotalAllocated - AvailableMoney)
```

The engine MUST NOT silently decide which user allocation to reduce. Until resolved, STS is `UNRESOLVED` rather than authoritative.

### `RULE-ALLOC-103` Flexible Money
Flexible Money is the explicit portion of actual Available Money the user has designated for discretionary spending. It is user intent, not a forecast and not identical to STS.

### `RULE-ALLOC-104` Reserved Money
Reserved Money remains liquid but is protected for a purpose. It contributes zero to ordinary STS unless explicitly released/reallocated. A reservation linked to an obligation represents funding already protected for that obligation and MUST NOT cause a second deduction.

### `RULE-ALLOC-105` Locked Money
Locked Money is intentionally unavailable for current spending. Technical withdrawability does not make it Flexible. It contributes zero to STS until explicitly unlocked/reallocated.

## 7. Income Allocation

### `RULE-INCOME-ALLOC-001`
A realized income transaction first increases the receiving asset balance/Available Money. It MUST NOT be auto-classified as Flexible, Reserved, or Locked by a hardcoded income source rule. In the absence of an explicit allocation command, the entire newly available delta remains Unallocated.

### `RULE-INCOME-ALLOC-002`
The product MUST allow the user to allocate newly received income to Flexible, Reserved, Locked, or leave it Unallocated. Previous choices MAY be suggested but do not execute automatically in V1.

## 8. Spending Budget Engine

A Budget is a cycle-scoped spending plan, separate from money allocation.

### `RULE-BUD-101` Budget spending
For budget `b` and its Financial Cycle `c`:

```text
BudgetSpent(b,c)
= Σ recognized actual EXPENSE amounts
  assigned to categories mapped to b
  with effective date inside c
```

**Exclusions:** transfers, liability payments for purchases already recognized as expenses, income, future/expected spending, voided transactions, refunds to the extent they reverse eligible budget spending.

### `RULE-BUD-102` Remaining and overspend

```text
BudgetRemaining = BudgetAmount - BudgetSpent
BudgetRemainingDisplayed = max(0, BudgetRemaining)
BudgetOverspend = max(0, BudgetSpent - BudgetAmount)
```

A negative raw remaining value is preserved semantically as overspend.

### `RULE-BUD-103` Utilization
If `BudgetAmount > 0`:

```text
BudgetUtilizationPct = BudgetSpent / BudgetAmount × 100
```

If `BudgetAmount = 0` and `BudgetSpent = 0`, utilization is `0%`. If `BudgetAmount = 0` and `BudgetSpent > 0`, status is `OVERSPENT_UNBUDGETED`; percentage is not mathematically presented as finite.

### `RULE-BUD-104` Warning status
Default V1 thresholds:

```text
0% <= utilization < 80%    = ON_TRACK_LIMIT
80% <= utilization < 100%  = APPROACHING_LIMIT
utilization = 100%          = LIMIT_REACHED
utilization > 100%          = OVERSPENT
```

Thresholds are constants/configuration-ready but not user-configurable in V1.

### `RULE-BUD-105` Expense accounting interaction
An expense changes financial reality once through its transaction/account effect and changes budget consumption as a classification of that same expense. **Budget consumption MUST NOT subtract cash a second time.**

If an expense is paid from actual Flexible-funded money, Flexible Money is reduced by the realized spending amount, subject to explicit allocation reconciliation. Budget overspend therefore reflects that Flexible has already been consumed; it is not an additional deduction.

### `RULE-BUD-106` Cycle close
Budgets do not financially roll over. At cycle close, the old Budget remains historical. Any unused budget amount does not create cash; the underlying unspent Flexible Money simply remains Flexible. The new cycle starts with no Budget instances unless the user creates/applies them.

### `RULE-BUD-107` Previous-cycle template
“Use previous cycle as template” copies budget definitions/amounts/category mappings into **new Budget instances** for the new cycle. It does not move money and does not copy Spent.

### `RULE-BUD-108` Budget is not reservation
Creating, increasing, decreasing, or deleting a Budget MUST NOT mutate Flexible, Reserved, Locked, Unallocated, Available Money, account balances, or STS by itself. A Budget is a spending plan only. Actual eligible expenses affect financial state once through Transaction/Account semantics and separately update Budget Spent.

### `RULE-BUD-109` Budget capacity warning
For active Budgets in the active Financial Cycle:

```text
TotalActiveBudgetAmount = Σ BudgetAmount(b)
BudgetPlanOverFlexible = max(0, TotalActiveBudgetAmount - FlexibleMoney)
```

`TotalActiveBudgetAmount > FlexibleMoney` is valid. It MUST NOT block Budget creation/update solely for this reason and MUST NOT reserve money. When `BudgetPlanOverFlexible > 0`, surface warning state `BUDGET_EXCEEDS_FLEXIBLE` with the exact difference.

**Recalculate:** Budget create/edit/delete/cancel, Flexible allocation change, newly realized transaction that changes allocation state, allocation-shortfall resolution, and cycle transition.

## 9. Financial Cycle

### `RULE-CYCLE-101` Configuration
A Financial Cycle has a recurring anchor day `D` and adjustment policy:

```text
NONE
PREVIOUS_WORKING_DAY
NEXT_WORKING_DAY
```

V1 working day means Monday–Friday in Financial Timezone. No public-holiday calendar is assumed.

### `RULE-CYCLE-102` Nominal anchor
For each month, nominal anchor date is day `min(D, last_day_of_month)`.

### `RULE-CYCLE-103` Adjustment
- `NONE`: effective anchor = nominal anchor.
- `PREVIOUS_WORKING_DAY`: if nominal anchor is Saturday/Sunday, move backward day-by-day to Friday; otherwise unchanged.
- `NEXT_WORKING_DAY`: if nominal anchor is Saturday/Sunday, move forward day-by-day to Monday; otherwise unchanged.

Do not hardcode “Friday” or “Monday” as rules; they are consequences of weekday evaluation.

### `RULE-CYCLE-104` Boundaries
A cycle starts at local `00:00:00` on one effective anchor and ends immediately before the next effective anchor. Date presentation is inclusive: start date through calendar date immediately preceding next start.

### `RULE-CYCLE-105` Active cycle
At evaluation instant `t`, active cycle is the unique cycle where `start <= t < next_start`.

### `RULE-CYCLE-106` Anchor or adjustment-policy change
Financial Calendar is the sole authority for Financial Cycle boundaries. Changing the anchor or adjustment policy MUST NOT retroactively redefine the active cycle. Determine the transition from the boundary immediately following the active cycle to the first regular boundary under the new anchor and policy. If the transition duration is less than 15 days, it MUST NOT become a standalone cycle; merge it with the following regular cycle as an Extended Cycle. If it is 15 days or more, create a standalone Short Cycle before regular cycles under the new configuration.

## 10. Obligations and Applicable Upcoming Obligations

### `RULE-OBL-101` Remaining due

```text
RemainingDue(o) = max(0, AmountDue - AmountSettled)
```

### `RULE-OBL-102` STS applicability
An obligation is applicable to current STS when it is unsettled and its due instant is **on or before the end of the active Financial Cycle**, or it is already overdue. Future Expected Income never removes this requirement.

### `RULE-OBL-103` Reserved coverage

```text
ApplicableReservedCoverage(o) = min(usable linked Reserved amount, ApplicableObligationAmount(o))
UnfundedObligationAmount(o) = max(0, ApplicableObligationAmount(o) - ApplicableReservedCoverage(o))
```

### `RULE-OBL-104` Upcoming obligations

```text
UpcomingObligations = Σ RemainingDue(o)
```

for unsettled obligations inside the requested planning horizon. For STS use the active-cycle applicability rule and unfunded amount.

## 11. Safe to Spend

### `RULE-STS-101` Definition
STS is current actual Flexible Money that can be spent while retaining enough actual Flexible funding for all applicable unfunded obligations in the active Financial Cycle.

If allocation state is valid:

```text
ApplicableUnfundedObligations
= Σ UnfundedObligationAmount(o)

RawSTS
= FlexibleMoney - ApplicableUnfundedObligations

DisplayedSTS
= max(0, RawSTS)

FundingDeficit
= max(0, -RawSTS)
```

### `RULE-STS-102` Exclusions
STS MUST NOT be increased by Expected Income, forecasted refunds, projected savings, unused credit, Unallocated Money, Reserved Money, or Locked Money.

### `RULE-STS-103` Allocation shortfall
If `AllocationShortfall > 0`, authoritative STS state is `UNRESOLVED`. The UI MAY show a diagnostic provisional calculation but MUST NOT label it authoritative Safe to Spend.

### Example

```text
Available actual liquid money      Rp6,800,000
Flexible allocation                Rp3,800,000
Reserved                           Rp1,000,000
Locked                             Rp2,000,000
Applicable obligation              Rp1,550,000
Reserved linked to obligation        Rp500,000
Unfunded applicable obligation     Rp1,050,000
Expected salary tomorrow          Rp10,000,000  (forecast only)

Raw STS = 3,800,000 - 1,050,000 = Rp2,750,000
```

The expected salary contributes **Rp0** to current STS.

## 12. Daily Safe to Spend

### `RULE-DSTS-101` Remaining days
`RemainingCycleDays` is the number of local calendar dates from the evaluation date through the final date of the active cycle, **including today**.

### `RULE-DSTS-102` Formula
If authoritative STS exists and `DisplayedSTS > 0`:

```text
DailySTS = floor_to_minor_unit(DisplayedSTS / RemainingCycleDays)
```

If `DisplayedSTS <= 0`, `DailySTS = 0`. If STS is unresolved, Daily STS is unresolved.

### `RULE-DSTS-103` Boundary
On the final cycle date, `RemainingCycleDays = 1`. At the next anchor, a new cycle is resolved before Daily STS calculation.

## 13. Budget Pace and Projection

### `RULE-PACE-101` Cycle progress

```text
TotalCycleDays = count of local calendar dates in active cycle
ElapsedCycleDays = count from cycle start date through evaluation date, including today
CycleProgressPct = ElapsedCycleDays / TotalCycleDays × 100
```

### `RULE-PACE-102` Budget consumption progress
For `BudgetAmount > 0`:

```text
BudgetConsumptionPct = BudgetSpent / BudgetAmount × 100
```

### `RULE-PACE-103` Pace state
With tolerance `PACE_TOLERANCE = 5 percentage points`:

```text
BudgetConsumptionPct <= CycleProgressPct + 5pp  => ON_PACE
BudgetConsumptionPct >  CycleProgressPct + 5pp  => AHEAD_OF_PLAN
```

Budget limit statuses from `RULE-BUD-104` remain independently applicable.

### `RULE-PROJ-BUD-101` Projected budget spending

```text
MIN_PROJECTION_ELAPSED_DAYS = 3
```

Budget utilization, Cycle Progress, and Pace may be calculated from Day 1. Projected Budget Spending and Projected Budget Overspend MUST NOT be displayed when `ElapsedCycleDays < 3`. The projection status is `NOT_YET_ELIGIBLE`.

When `ElapsedCycleDays >= 3` and transaction history is complete enough for the Budget:

```text
AverageDailySpend = BudgetSpent / ElapsedCycleDays
ProjectedBudgetSpending
= floor_to_minor_unit(AverageDailySpend × TotalCycleDays)
ProjectedBudgetOverspend
= max(0, ProjectedBudgetSpending - BudgetAmount)
```

If `BudgetSpent = 0` on or after Day 3, `ProjectedBudgetSpending = 0` and `ProjectedBudgetOverspend = 0`. If transaction history for the budget is known incomplete/unknown, projection status is `INSUFFICIENT_DATA` and no numeric projection is shown. A Budget created after Day 3 uses the active cycle's `ElapsedCycleDays`, not the Budget's age; its `BudgetSpent` still derives from all eligible mapped actual expenses in that cycle.

**Recalculate:** relevant transaction create/edit/void/backdate/refund/recategorization, Budget create/edit/delete, and financial date/cycle transition.

### `RULE-PROJ-BUD-102` Daily spending to return on track

```text
FutureDays = TotalCycleDays - ElapsedCycleDays
RemainingBudget = max(0, BudgetAmount - BudgetSpent)
```

If `FutureDays > 0` and `RemainingBudget > 0`:

```text
MaxAverageDailySpendToStayWithinBudget
= floor_to_minor_unit(RemainingBudget / FutureDays)
```

If already overspent, this value is `0`; the advisor MUST NOT imply the historical overspend can be undone.

## 14. Forecast Engine

### `RULE-FCST-101` Actual starting point
Forecast starts from a snapshot of actual financial state at evaluation time. Forecast events MUST NOT mutate canonical actual state.

### `RULE-FCST-102` Base projection
The **Base Forecast** has deterministic inclusion semantics:

```text
BaseProjectedBalance(t)
= ActualStartingLiquidBalance
+ Σ EXPECTED_STABLE income with expected_time <= t
+ Σ other deterministic future cash inflows explicitly modeled by existing V1 rules
- Σ scheduled future cash outflows through t
```

`EXPECTED_UNCERTAIN` income is excluded from Base Forecast. Expected Income remains a projection assumption and never mutates actual state or STS.

### `RULE-FCST-103` Scenario Forecast
A Scenario Forecast starts from Base Forecast and adds only the explicitly selected `EXPECTED_UNCERTAIN` events:

```text
ScenarioProjectedBalance(t, S)
= BaseProjectedBalance(t)
+ Σ expected amount(e)
  for selected EXPECTED_UNCERTAIN event e in scenario S
  where expected_time(e) <= t
```

If no uncertain events are selected, Scenario Forecast equals Base Forecast. Scenario inclusion MUST be visible/explainable and MUST NOT mutate actual state.

### `RULE-FCST-103A` Expected Income state transition
Income Management owns Expected Income lifecycle; Financial Accounts & Ledger owns actual Income. An occurrence remains `EXPECTED` until cancelled, its expected time passes without an explicit link (`MISSED`), or the user explicitly links a posted actual Income transaction (`RECEIVED`). The link is optional, so actual Income may remain unlinked. Amount/date/description/source similarity, date arrival, or another heuristic MUST NOT create the link or mark the occurrence `RECEIVED`. For date-only expectations, "passed" means after the end of that local financial date. A late actual receipt MAY move `MISSED` to `RECEIVED` only through the same explicit user link; actual amount/time remains authoritative.

### `RULE-FCST-104` Projected cycle-end balance
Evaluate `ProjectedBalance` immediately before the next Financial Cycle start. Actual/assumed components MUST be explainable.

### `RULE-FCST-105` No double counting
Liability purchase spending and later liability payment are distinct expense-vs-cash-flow stages. Forecast cash outflow includes the scheduled payment; spending analytics/budgets recognize the purchase expense, not payment again.

### `RULE-FCST-106` Funding Dependency / Boundary Risk
Funding Dependency is a Forecast/Advisor condition; it never modifies STS. Evaluate each future unsettled obligation `o` in the Planning horizon, especially obligations after the current-cycle boundary.

Define the actual-only projected liquid balance immediately before `o` while excluding all Expected Income:

```text
ActualOnlyBalanceBefore(o)
= ActualStartingLiquidBalance
+ Σ modeled future actual/non-income cash inflows before o
- Σ scheduled cash outflows before o
```

Define required cash for `o` as its forecast cash outflow after any funding already represented by actual protected/reserved money according to existing obligation rules.

A Funding Dependency exists when:

```text
RequiredCash(o) > max(0, ActualOnlyBalanceBefore(o))
AND
Σ EXPECTED_STABLE income scheduled before o > 0
```

The dependency amount is:

```text
FundingDependencyAmount(o)
= min(RequiredCash(o),
      max(0, RequiredCash(o) - max(0, ActualOnlyBalanceBefore(o))))
```

The Forecast MUST also show whether Base Forecast is still insufficient even after stable Expected Income; in that case both Funding Dependency and a projected funding deficit/shortfall may exist.

Funding Dependency disappears when any recalculation makes `RequiredCash(o) <= max(0, ActualOnlyBalanceBefore(o))`, the obligation is settled/cancelled, or no stable Expected Income remains before `o`. If a previously supporting Expected Income becomes `MISSED` or moves after the obligation, the dependency is replaced by the appropriate projected funding-risk/deficit state rather than being treated as funded.

**Recalculate:** transaction create/edit/void/backdate, explicit Expected Income link/unlink, allocation/reservation change affecting obligation funding, obligation create/edit/pay/cancel, Expected Income create/edit/receive/miss/cancel, liability schedule change, and time/cycle transition.

## 15. Credit Card Engine

### `RULE-CC-101` Purchase recognition
A credit-card purchase recognizes actual Expense at purchase time and increases Liability Outstanding by principal/recognized purchase amount. It does not reduce an asset account at purchase time.

### `RULE-CC-102` Statement/cutoff
Purchases are assigned to statement periods using configured cutoff semantics. Statement amount due creates/updates an Obligation with configured due date.

### `RULE-CC-103` Payment
Payment reduces the paying asset and Liability Outstanding and settles the obligation. It MUST NOT create a second category expense.

### `RULE-CC-104` Installment conversion
Conversion supersedes the original future repayment representation and creates the actual installment schedule without duplicating purchase expense/principal.

## 16. BNPL / Installments / Fees

### `RULE-INST-101` BNPL purchase
Purchase expense is recognized at purchase time. Liability includes purchase principal plus separately recognized fees/interest that the user is obligated to pay.

### `RULE-INST-102` Schedule
Each installment has explicit due date and **actual required installment amount**. The engine MUST NOT assume `principal / count` when fees/interest/provider terms produce a different amount.

### `RULE-INST-103` Fees/interest
Known purchase-time fees may be recognized as a separate fee expense and liability when incurred. Financing interest/fees that accrue later are recognized as expense/liability when contractually incurred according to the entered schedule. Payment itself is not a second expense.

## 17. Recurring and Staged Obligations

### `RULE-REC-101` Recurring rule
A Recurring Obligation Rule generates obligation instances from recurrence and effective periods. Generated instances are canonical obligations for settlement/forecast/STS.

### `RULE-REC-102` Staged amount
A rule MAY contain non-overlapping effective amount periods, e.g.:

```text
2026-10-01..2028-09-30  Rp3,200,000/month
2028-10-01..2030-09-30  Rp3,750,000/month
2030-10-01..open         Rp4,100,000/month
```

For an occurrence date, exactly one applicable stage MUST resolve. Overlapping stages are invalid.

## 18. Deterministic Advisor / Insights

### `RULE-ADV-101` Inputs only
Advisor numeric statements MUST be generated only from Financial Engine derived values. It MUST NOT invent, estimate with an LLM, or silently alter financial numbers.

### `RULE-ADV-102` Insight structure
A financial insight SHOULD map deterministic data into:

```text
WHAT HAPPENED → WHY IT MATTERS → LIKELY OUTCOME → IMPACT → POSSIBLE ACTION
```

### `RULE-ADV-103` Budget pace insight
When a budget is `AHEAD_OF_PLAN`, the advisor MAY state cycle progress, budget consumption, Projected Budget Spending/Overspend, and `MaxAverageDailySpendToStayWithinBudget` when available.

### `RULE-ADV-104` Funding Dependency insight
When `RULE-FCST-106` detects Funding Dependency, the Advisor MAY explain the future obligation, the actual-only funding shortfall before that obligation, the `EXPECTED_STABLE` income scheduled before it, and whether Base Forecast remains deficient after that income. This insight MUST NOT change current STS. If the supporting income becomes `MISSED` or moves after the obligation, the Advisor MUST stop describing the obligation as dependent on that income and surface the resulting projected funding risk/deficit.

### `RULE-ADV-105` No autonomous action
Advisor suggestions do not move money, edit budgets, change income assumptions, or change obligations without explicit user action.

## 19. Recalculation Triggers

Recalculate affected derived values after, as applicable:

- transaction create, edit, void/delete-equivalent, backdate, refund, or recategorization;
- actual Income receipt and explicit Expected Income link/unlink;
- allocation or reallocation change;
- account balance or liquidity-eligibility change;
- obligation create, edit, settle/pay, cancel, or due-time change;
- reservation link, amount change, release, or reallocation;
- Budget create, edit, cancel/delete-equivalent, template application, or category mapping change;
- Expected Income create, edit, certainty change, explicitly link as `RECEIVED`, unlink, cancel, or transition to `MISSED`;
- recurring obligation generation/stage edit;
- liability, credit statement, BNPL, or installment schedule mutation;
- Financial Cycle configuration change, local financial date transition relevant to elapsed/remaining days, or cycle boundary transition.

Corrections MUST propagate to balances, allocations/shortfall, Budgets, STS/Daily STS, Base/Scenario Forecast, Funding Dependency, projected deficits, and Advisor insights as applicable. Implementations MAY optimize dependency graphs but MUST produce the same logically consistent final state as full recomputation.

## 20. Explainability Contract

For each important derived value the engine MUST expose or be able to reconstruct:
- definition and evaluation time;
- input records/amounts;
- exclusions;
- formula/rule IDs;
- rounding;
- status/assumptions;
- recalculation provenance sufficient for debugging.

Forecast explanations MUST label actual starting state separately from assumed future events.

## 21. Acceptance Invariants

- `INV-FE-001`: Expected Income contributes zero to current STS before an explicitly linked actual receipt marks it `RECEIVED`.
- `INV-FE-002`: Transfers between owned accounts are not spending.
- `INV-FE-003`: Credit/BNPL payment never duplicates purchase expense.
- `INV-FE-004`: Budget consumption never deducts cash a second time.
- `INV-FE-005`: Locked and Reserved money are excluded from ordinary STS.
- `INV-FE-006`: Unused budget does not create money at cycle close.
- `INV-FE-007`: New-cycle budget Spent starts at zero; account/liability balances persist.
- `INV-FE-008`: All monetary outputs use integer minor units and deterministic rounding.
- `INV-FE-009`: Financial Cycle weekday adjustment is deterministic without a holiday dependency.
- `INV-FE-010`: Every advisor number is traceable to an engine output.
- `INV-FE-011`: Newly available money never becomes Flexible without explicit allocation.
- `INV-FE-012`: Budget amount does not reserve money and may exceed Flexible with a warning.
- `INV-FE-013`: Obligations after the active-cycle boundary do not reduce current STS.
- `INV-FE-014`: Numeric budget projection is suppressed on Cycle Day 1 and Day 2.
- `INV-FE-015`: Funding Dependency is informational/forecast-derived and never modifies actual state or STS.

## 22. Final Closure Invariants

- `RULE-CLOSE-001`: newly available money remains Unallocated until an explicit allocation command changes its role.
- `RULE-CLOSE-002`: Unallocated contributes zero to STS.
- `RULE-CLOSE-003`: Budget creation/amount changes never reserve or move money; `TotalActiveBudgetAmount > FlexibleMoney` is valid with warning.
- `RULE-CLOSE-004`: STS considers only applicable unfunded obligations due in or overdue into the active Financial Cycle; post-cycle obligations do not reduce current STS.
- `RULE-CLOSE-005`: `EXPECTED_STABLE` enters Base Forecast; `EXPECTED_UNCERTAIN` enters only selected Scenario Forecasts; neither increases current STS.
- `RULE-CLOSE-006`: numeric Budget projection is unavailable before 3 elapsed cycle days.
- `RULE-CLOSE-007`: Funding Dependency is derived from Forecast primitives and never mutates Actual state.
