# Financial Rules

This extraction preserves source IDs, formulas, thresholds, examples and qualifications for review. This document and the other living references are the active baseline; archived specifications preserve historical context. Repeated definitions and examples are deliberately retained where they aid comparison; no SQL schema or implementation is implied. Links point to the archived originals.

**Unresolved behavior:** The Ledger baseline is owner-frozen with open findings, not independently verified or implementation-complete. The Ledger section reproduces current `AUD-LDG-001`–`009` dispositions; see also `Q-04`–`06` in [DECISIONS.md](DECISIONS.md). Do not infer missing inputs, ordering or accounting treatment. Earlier candidate wording is historical where superseded by frozen Ledger §29.

**Reading the examples:** Amounts and rounding are reproduced as written in each source, with no inferred currency precision. The Ledger “truthful shortfall” example assumes allocations remain unchanged; the engine separately says Flexible-funded expenses reduce Flexible subject to reconciliation. Their interaction requires explicit reconciliation under `Q-05` / `AUD-LDG-003`; do not silently treat the example as defining every expense allocation.


## Core Definitions & Money/Time Conventions

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 1. Purpose and Engine Separation

V1 has two strictly separated calculation contexts:

```text
ACTUAL / SAFETY ENGINE                    FORECAST ENGINE
realized balances                         starts from actual state
+ realized transactions                   + expected future income
+ explicit allocations                    - future obligations/outflows
+ applicable obligations                  ± deterministic future events
= current financial guidance              = projected future position
```

`RULE-STATE-001`: A future Expected Income assumption MUST NOT itself increase Account Balance, Available Money, Flexible Money, Budget Remaining, Raw STS, Displayed STS, or Daily STS. A posted actual Income affects actual account state whether or not the user explicitly links an Expected Income occurrence. Only an explicit link marks an eligible occurrence `RECEIVED`; actual newly available money remains Unallocated unless explicitly allocated. This wording reconciles `FR-TRX-002`, `FR-INC-003` and `RULE-FCST-103A` without changing their approved behavior.

`RULE-STATE-002`: Forecast outputs MUST identify assumptions separately from actual facts.

### 2. Monetary Arithmetic

#### `RULE-MONEY-001` Integer minor units
All canonical money is stored/calculated as integer minor units. Floating point MUST NOT be used.

#### `RULE-MONEY-002` Domain amounts
Canonical entered amounts are non-negative; direction comes from transaction/event semantics.

#### `RULE-MONEY-003` Division and rounding
Unless a rule states otherwise, division-derived spend guidance is rounded **down toward zero** to the currency minor unit. Percentages MAY retain implementation precision but displayed percentages MUST use a documented consistent rounding method.

### 3. Time Model

#### `RULE-TIME-001` Financial timezone
All cycle boundaries, date-only events, “today”, and elapsed/remaining-day calculations use the configured Financial Timezone.

#### `RULE-TIME-002` Date-only semantics
A date-only posted transaction belongs to that local financial date. A date-only obligation is due by the end of that local date. Expected Income timing affects Forecast only.

#### `RULE-TIME-003` Explicit timestamp
When a timestamp is supplied, use its resolved instant in the Financial Timezone.

> Source: [docs/archive/design/08-domain-model.md](docs/archive/design/08-domain-model.md). Source IDs and numeric examples are retained.

### 1. Principles

- `DM-PRINCIPLE-001` One economic event MUST affect aggregate financial state exactly once.
- `DM-PRINCIPLE-002` Actual and Forecast are separate states; forecast records never masquerade as realized transactions.
- `DM-PRINCIPLE-003` Allocation classifies owned money; it does not create money.
- `DM-PRINCIPLE-004` Budget is a spending plan; it is not a money allocation.
- `DM-PRINCIPLE-005` Credit capacity is liability capacity, not liquidity.
- `DM-PRINCIPLE-006` Derived values are recomputable, not independently editable.
- `DM-PRINCIPLE-007` This is a conceptual model, not a mandated physical database schema.

> Source: [docs/archive/design/08-domain-model.md](docs/archive/design/08-domain-model.md). Source IDs and numeric examples are retained.

### 3. Account — `DM-ACCOUNT-001`

Fields: `account_id`, `name`, `account_class ASSET|LIABILITY`, generic `account_type`, `currency`, `status ACTIVE|ARCHIVED`, `liquidity_eligible`, opening state, timestamps.

Invariants:
- liability credit limit contributes zero Available Money;
- archive preserves history/economic effect;
- an asset opening balance is an effective-time snapshot; pre-snapshot transactions remain historical facts but do not apply again to that snapshot or subsequent balance (`FR-LDG-012`);
- provider names are configuration/data, not engine logic.

### 4. Transaction — `DM-TRANSACTION-001`

A realized economic event.

Kinds:

```text
INCOME
EXPENSE
TRANSFER
LIABILITY_PURCHASE
LIABILITY_PAYMENT
REFUND
BALANCE_ADJUSTMENT
FEE_INTEREST
```

Required semantics: immutable ID, amount `>0`, effective time, currency, status `POSTED|VOIDED`, affected account leg(s), optional category, provenance.

Invariants:
- transfers between owned accounts are not income/expense;
- liability purchase recognizes spending and liability at purchase time;
- liability payment reduces asset + liability without duplicate expense;
- void/edit propagates to all dependent derived values.

### 5. Category — `DM-CATEGORY-001`

A user/system-defined spending purpose used to classify Expense/Fee transactions and map them to Budgets. V1 MUST NOT hardcode the founder's category list as the only supported taxonomy.

### 6. Allocation — `DM-ALLOCATION-001`

Types: `FLEXIBLE | RESERVED | LOCKED`. `UNALLOCATED` is derived.

Fields: ID, type, amount, purpose, status, optional linked obligation, optional source account constraint.

```text
Flexible + Reserved + Locked + Unallocated = Available Money
```

Budget is explicitly **not** an Allocation type. Newly available money does not create an Allocation record; with explicit allocations unchanged, it increases derived Unallocated. Only explicit allocation commands move money among intent roles.

### 7. Financial Cycle — `DM-CYCLE-001`

Configuration/domain instance describing cycle boundaries.

Fields/semantics:
- `anchor_day` 1..31;
- `adjustment_policy NONE|PREVIOUS_WORKING_DAY|NEXT_WORKING_DAY`;
- Financial Timezone;
- derived start/next-start/end dates.

Financial Calendar is the sole boundary authority. An anchor or adjustment-policy change never rewrites the active cycle. Its post-active-cycle transition is an Extended Cycle when shorter than 15 days (merged with the following regular cycle), or a standalone Short Cycle when at least 15 days.

V1 working-day adjustment uses weekdays only; no holiday-calendar entity is required.

### 8. Budget — `DM-BUDGET-001`

A cycle-scoped spending plan.

Fields:
- `budget_id`;
- `financial_cycle_id`;
- `name/purpose`;
- `budget_amount >= 0`;
- status `ACTIVE|CLOSED|CANCELLED`;
- one or more category mappings;
- creation/template provenance.

Derived: `spent`, `remaining`, `overspend`, `utilization`, pace state, projected spending/overspend.

Invariants:
- each Budget belongs to exactly one Financial Cycle;
- Spent derives only from actual eligible expenses;
- budget consumption does not move money;
- new-cycle template creates new Budget IDs and Spent=0;
- old budgets remain historical;
- Budget creation/amount changes do not reserve or move money;
- aggregate Budget amount is not constrained by Flexible Money; an excess is a derived warning state, not invalid financial state.

### 9. Obligation — `DM-OBLIGATION-001`

A known required outflow.

Fields: ID, source type, source ID, amount due, amount settled, due time, status, optional linked reservation.

Sources include `MANUAL|CREDIT_STATEMENT|BNPL|INSTALLMENT|RECURRING_RULE`.

`remaining_due=max(0, amount_due-amount_settled)`. Overdue unpaid obligations remain active.

### 10. Recurring Obligation Rule — `DM-REC-OBL-001`

Defines predictable obligation generation.

Fields: ID, recurrence pattern, start/end, due-day/timing semantics, status, one or more non-overlapping `AMOUNT_STAGE` records with `effective_from`, optional `effective_to`, and amount.

Exactly one amount stage must resolve for each generated occurrence. This supports staged KPR/subscription/payment amounts without assuming constant amount forever.

### 11. Expected Income — `DM-EXPECTED-INCOME-001`

A future inflow assumption owned by Income Management, never actual money before an explicitly linked actual receipt.

Fields: ID, expected amount/date, recurrence/source, `certainty EXPECTED_STABLE|EXPECTED_UNCERTAIN`, status `EXPECTED|RECEIVED|MISSED|CANCELLED`, optional actual Income transaction link. `EXPECTED_STABLE` has deterministic Base Forecast inclusion; `EXPECTED_UNCERTAIN` is Scenario-only unless explicitly selected.

Invariants:
- contributes zero to Account Balance, Available, Flexible, STS, and Daily STS before an explicitly linked actual receipt;
- may participate in Forecast/scenarios;
- `EXPECTED→RECEIVED` requires the user's explicit link to an actual INCOME transaction owned by Financial Accounts & Ledger and retires the future duplicate effect; the resulting newly available money remains Unallocated until explicit allocation;
- actual Income may remain unlinked; amount/date/description/source similarity cannot mark Expected Income `RECEIVED`;
- if expected time passes without linked receipt, status becomes `MISSED`; a late explicit link may transition `MISSED→RECEIVED`;
- no V1 probability/confidence score.

### 12. Credit Statement — `DM-CREDIT-STMT-001`

Statement-period representation for revolving liability: account, period, cutoff, due date, amount due, status, linked obligation. Purchase expense remains recognized at purchase time; statement is repayment planning, not a second expense.

### 13. Installment Plan / Installment — `DM-INSTALLMENT-001`

Plan fields include source liability/purchase, principal, known total fees/interest where applicable, status. Each Installment has due date and **actual required installment amount** and normally creates one obligation. The model MUST NOT derive installment amount solely as principal/count unless that is explicitly the contractual schedule.

### 14. Settlement Link — `DM-SETTLEMENT-001`

Links a posted payment transaction to an Obligation and settlement amount. Total valid settlements cannot exceed obligation amount due.

### 15. Forecast Projection — `DM-FORECAST-001`

Forecast is a derived projection, not a canonical money container. A projection consists of:
- actual starting snapshot;
- included future events;
- assumption/scenario set;
- projected positions over time;
- engine/rule version.

Forecast values MUST NOT be written back as actual balances. Derived forecast states include Base Forecast, selected Scenario Forecasts, projected funding deficit/risk, and Funding Dependency. Funding Dependency is not a canonical money container and does not alter STS.

### 16. Advisor Insight — `DM-INSIGHT-001`

A derived explanatory object containing insight type, referenced engine metrics, explanation sections, and optional deterministic corrective amount. It has no authority to mutate financial state.

### 17. Core Separation

```text
ACCOUNT + TRANSACTION              = actual financial reality
ALLOCATION                         = role of actual owned money
BUDGET                             = planned spending limit for a cycle
OBLIGATION                         = required outflow
EXPECTED_INCOME                    = future assumption
FORECAST                           = projected future state
STS / DAILY STS                    = current safety guidance
ADVISOR INSIGHT                    = deterministic interpretation
```

> Source: [docs/archive/reference/11-glossary.md](docs/archive/reference/11-glossary.md). Source IDs and numeric examples are retained.

### Account
A tracked financial container classified as **Asset** or **Liability**.

### Asset
An owned economic resource. Only positive balances of liquidity-eligible Asset accounts participate in Available Money.

### Liability
An amount owed. Credit limits/unused borrowing capacity are not owned money and contribute zero Available Money.

### Total Balance
A display aggregate of selected account balances. Because assets and liabilities have different semantics, “Total Balance” MUST NOT be used as a synonym for Available Money or STS.

### Actual Financial State
Financial state derived only from events that have occurred: posted transactions, actual account/liability balances, existing allocations, and known obligations. Expected Income is not Actual Financial State.

### Forecast Financial State
A projected state starting from Actual Financial State and applying explicit future assumptions/events. It does not mutate Actual Financial State.

### Available Money
Actual positive owned liquid money eligible for allocation:

```text
AvailableMoney = Σ positive liquidity-eligible Asset balances
```

Expected Income and credit limits are excluded.

### Flexible Money
The explicit portion of Available Money the user has designated for discretionary spending. Flexible is user intent; it is not automatically equal to STS.

### Reserved Money
Actual liquid money protected for a designated purpose. It normally contributes zero to STS. A reservation may fund an Obligation without causing a second deduction.

### Locked Money / Locked Savings
Actual owned money intentionally excluded from current spending. Technical withdrawability does not make it safe to spend.

### Unallocated Money

```text
Unallocated = Available - Flexible - Reserved - Locked
```

Money whose role has not yet been explicitly assigned. Newly available money—including starting balance and received income—remains Unallocated until an explicit allocation command changes its role. It contributes zero to STS.

### Spendable Money
For V1 documentation, use **Flexible Money** when referring to user-designated discretionary money and **Safe to Spend** when referring to engine safety guidance. “Spendable Money” SHOULD NOT be used as a separate derived metric because it creates ambiguity.

### Money Allocation
Classification of actual owned money by role: Flexible, Reserved, Locked, or derived Unallocated. Allocation answers: **“What role does this money currently have?”**

### Category
A spending purpose used to classify actual expenses and associate them with Budgets.

### Budget
A Financial-Cycle-scoped spending plan for one purpose/category set. Budget answers: **“How much do I plan to spend for this purpose in this cycle?”** It is not an Allocation or reservation. Creating/changing a Budget moves no money. Total Active Budget may exceed Flexible Money; this is valid and produces a non-blocking warning.

### Budget Amount
The planned spending limit for one Budget instance in one Financial Cycle.

### Budget Spent
Sum of actual eligible expense transactions mapped to the Budget during its Financial Cycle. Transfers and liability repayments already represented by purchase expenses are excluded.

### Budget Remaining

```text
BudgetRemaining = BudgetAmount - BudgetSpent
```

Negative value means overspending; display may separately show `BudgetOverspend=max(0,-BudgetRemaining)`.

### Budget Utilization
For positive Budget Amount:

```text
BudgetUtilizationPct = BudgetSpent / BudgetAmount × 100
```

### Budget Spending Pace
Comparison of Budget Utilization against elapsed Financial Cycle percentage. V1 considers consumption more than 5 percentage points ahead of cycle progress `AHEAD_OF_PLAN`.

### Projected Budget Spending
Deterministic run-rate projection:

```text
ProjectedBudgetSpending
= (BudgetSpent / ElapsedCycleDays) × TotalCycleDays
```

rounded down to minor units. It is not an AI prediction. The numeric projection is not eligible on Cycle Day 1 or Day 2; it becomes eligible when `ElapsedCycleDays >= 3` and required history is sufficiently complete.

### Financial Cycle
Canonical budgeting/safety period defined by recurring anchor day and adjustment policy. A payday-based cycle is one configuration; calendar month is not assumed.

Adjustment policies: `NONE`, `PREVIOUS_WORKING_DAY`, `NEXT_WORKING_DAY`. V1 working day means Monday–Friday; public holidays are not modeled.

Financial Calendar is the sole boundary authority. Configuration changes do not redefine the active cycle. A post-active-cycle transition shorter than 15 days merges with the following regular cycle as an **Extended Cycle**; one lasting at least 15 days is a standalone **Short Cycle**.

### Obligation
A known required outflow with amount due, due date, settlement state, and source. Examples: KPR payment, bill, credit-card statement, BNPL payment, installment.

### Recurring Obligation
An obligation generated from a recurrence rule. Its amount may vary by effective period/stage.

### Expected Income
A future income assumption owned by Income Management. Canonical certainty is `EXPECTED_STABLE` or `EXPECTED_UNCERTAIN`; lifecycle states are `EXPECTED|RECEIVED|MISSED|CANCELLED`. `EXPECTED_STABLE` is included in Base Forecast; `EXPECTED_UNCERTAIN` enters only selected Scenario Forecasts. Expected Income contributes zero to actual-money and STS metrics. It becomes `RECEIVED` only through an optional explicit user link to an actual Income transaction owned by Financial Accounts & Ledger. An actual Income may remain unlinked; date arrival or amount/date/description/source similarity cannot mark it received. A late explicit link may move `MISSED` to `RECEIVED`.

### Actual Income
A posted INCOME transaction that has actually occurred. It changes actual account balance/Available Money. The newly available amount remains Unallocated until explicitly allocated by the user.

### Applicable Reserved Coverage
The usable Reserved Money explicitly protecting an applicable obligation, capped at that obligation's applicable amount.

### Unfunded Obligation Amount
`max(0, Applicable Obligation Amount - Applicable Reserved Coverage)`. STS deducts only this uncovered portion from Flexible, so funded Reserved coverage is not subtracted twice.

### Safe to Spend (STS)
Current actual Flexible Money remaining after protecting applicable unfunded obligations through the end of the active Financial Cycle:

```text
RawSTS = FlexibleMoney - ApplicableUnfundedObligations
DisplayedSTS = max(0, RawSTS)
FundingDeficit = max(0, -RawSTS)
```

Expected Income contributes zero before an actual receipt, regardless of its lifecycle label.

### Funding Deficit
Amount by which applicable current-cycle obligations exceed actual Flexible funding after reservation coverage.

### Daily Safe to Spend

```text
DailySTS = floor(DisplayedSTS / RemainingCycleDays)
```

RemainingCycleDays includes today through the last date of the active cycle. If STS <= 0, Daily STS = 0. If STS is unresolved due to allocation shortfall, Daily STS is unresolved.

### Forecast
A deterministic projection starting from actual current financial state and applying explicit future cash-flow assumptions. Forecast must distinguish actual facts from assumptions.

### Projected Balance
Projected liquid balance at a future instant under a specified Forecast assumption set.

### Projected Cycle-End Balance
Projected Balance immediately before the next Financial Cycle begins.

### Liability Outstanding
Actual amount currently owed on a liability after recognized purchases/principal/fees and actual payments/refunds.

### Advisor / Insight
A deterministic interpretation of engine outputs explaining what happened, why it matters, likely projected impact, and possible corrective action. It is not an LLM chatbot and cannot invent numeric values.

### Transfer
Movement between owned accounts. It changes account locations/balances but does not count as income or expense.

### Base Forecast
Derived forecast that starts from actual liquid state, includes `EXPECTED_STABLE` future income and scheduled future outflows, and excludes `EXPECTED_UNCERTAIN` income. It never mutates actual state.

### Scenario Forecast
A Base Forecast plus explicitly selected `EXPECTED_UNCERTAIN` events. It is derived and never changes STS or actual balances.

### Funding Dependency / Boundary Risk
A Forecast/Advisor condition where a future obligation lacks sufficient actual-only projected liquid funding before its due time and therefore depends on unreceived `EXPECTED_STABLE` income scheduled before it. It does not extend or modify the current-cycle STS horizon.

### Budget Plan Over Flexible
`max(0, TotalActiveBudgetAmount - FlexibleMoney)`. A positive value produces a non-blocking warning; it is not an allocation shortfall and does not reserve money.

### Projection Eligibility
Numeric Projected Budget Spending/Overspend is eligible only when at least 3 Financial Cycle days have elapsed and required transaction history is sufficiently complete.

### Canonical Terminology Rules
- `GLO-RULE-001`: Budget and Allocation are never synonyms.
- `GLO-RULE-002`: Flexible Money and STS are never synonyms.
- `GLO-RULE-003`: Expected Income and Actual Income are never synonyms; only an explicit user link marks the Expected Income `RECEIVED`.
- `GLO-RULE-004`: Forecast values are never presented as actual balances.
- `GLO-RULE-005`: Liability payment is not a duplicate expense.
- `GLO-RULE-006`: “Financial Cycle” is canonical; “payday cycle” describes one configuration.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 12. Dashboard & Navigation

#### `FR-NAV-001`
Primary navigation SHOULD remain approximately `Home | Transactions | Budget | Accounts | Planning` across platforms, adapting chrome without changing ownership of concepts.

#### `FR-HOME-001`
Home prioritizes STS/Funding Deficit, Daily STS, budget health/attention, upcoming obligations, and contextual insights.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-MONEY-001` | Fraction unsupported by currency minor unit | Reject invalid canonical amount; no floating-point storage. | REJECT |


## Safe-to-Spend & Available Money Formulas

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 5. Available Money

#### `RULE-AVAILABLE-001` Definition
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

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 11. Safe to Spend

#### `RULE-STS-101` Definition
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

#### `RULE-STS-102` Exclusions
STS MUST NOT be increased by Expected Income, forecasted refunds, projected savings, unused credit, Unallocated Money, Reserved Money, or Locked Money.

#### `RULE-STS-103` Allocation shortfall
If `AllocationShortfall > 0`, authoritative STS state is `UNRESOLVED`. The UI MAY show a diagnostic provisional calculation but MUST NOT label it authoritative Safe to Spend.

#### Example

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

### 12. Daily Safe to Spend

#### `RULE-DSTS-101` Remaining days
`RemainingCycleDays` is the number of local calendar dates from the evaluation date through the final date of the active cycle, **including today**.

#### `RULE-DSTS-102` Formula
If authoritative STS exists and `DisplayedSTS > 0`:

```text
DailySTS = floor_to_minor_unit(DisplayedSTS / RemainingCycleDays)
```

If `DisplayedSTS <= 0`, `DailySTS = 0`. If STS is unresolved, Daily STS is unresolved.

#### `RULE-DSTS-103` Boundary
On the final cycle date, `RemainingCycleDays = 1`. At the next anchor, a new cycle is resolved before Daily STS calculation.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 8. Safe to Spend

#### `FR-STS-001`
Home displays authoritative STS/Daily STS when allocation state is valid, plus Funding Deficit when positive. STS considers only unfunded obligations due within the active Financial Cycle (plus already-overdue obligations); obligations due after the cycle boundary do not reduce current STS.

#### `FR-STS-002`
“Why this amount?” shows actual Flexible, applicable obligations, reservation coverage, exclusions (including Expected Income), formula and cycle horizon.

#### `FR-STS-003`
Expected Income MUST NEVER be presented as increasing current STS before receipt.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-STS-001` | Raw STS<0 | Display STS=0 and Funding Deficit=abs(raw). | DERIVE |
| `EC-STS-002` | Flexible=0, Unallocated>0 | STS=0; Unallocated is not automatic spend permission. | DERIVE |
| `EC-STS-003` | Locked/Reserved could cover deficit | Do not automatically release. | ACCEPT + SURFACE |
| `EC-STS-004` | Allocation shortfall | Authoritative STS/Daily STS unresolved. | ACCEPT + SURFACE |
| `EC-DSTS-001` | Final cycle day | Remaining days=1. | DERIVE |
| `EC-STS-BND-001` | Obligation due final day of active cycle | It is STS-applicable. | DERIVE |
| `EC-STS-BND-002` | Obligation due one day after active cycle | It does not reduce current STS; Planning/Forecast evaluates future risk. | DERIVE |

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

#### `EC-NUM-001` Expected income never bridges STS

```text
Flexible actual                 Rp5,000,000
Expected salary                Rp10,000,000
Current-cycle obligation        Rp6,000,000
Raw STS                        -Rp1,000,000
Displayed STS                   Rp0
Funding Deficit                 Rp1,000,000
```

Forecast may show improved future position after salary, but current STS remains zero.


## Allocations, Budgets & Obligations

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 6. Money Allocation

#### `RULE-ALLOC-101` Explicit roles

```text
TotalAllocated = FlexibleMoney + ReservedMoney + LockedMoney
UnallocatedMoney = AvailableMoney - TotalAllocated
```

Normal user allocation mutations MUST satisfy `0 <= TotalAllocated <= AvailableMoney`. `UNALLOCATED` is derived; it is not an allocation record or implicit spend permission.

#### `RULE-ALLOC-101A` Newly available money
Any increase in actual Available Money for which no explicit allocation mutation occurs—including onboarding starting balance, received salary, received freelance income, refunds that create newly available liquidity, or other actual inflows—MUST leave existing Flexible/Reserved/Locked amounts unchanged. Therefore the increase appears in `UnallocatedMoney`. No system action may silently convert that delta to Flexible.

**Example:** starting Cash `Rp5,000,000`, no allocations → Flexible `Rp0`, Reserved `Rp0`, Locked `Rp0`, Unallocated `Rp5,000,000`, STS `Rp0`.

#### `RULE-ALLOC-102` Allocation shortfall after reality changes
A truthful realized transaction MAY cause `TotalAllocated > AvailableMoney`. The transaction MUST remain recorded. The system enters `ALLOCATION_SHORTFALL`:

```text
AllocationShortfall = max(0, TotalAllocated - AvailableMoney)
```

The engine MUST NOT silently decide which user allocation to reduce. Until resolved, STS is `UNRESOLVED` rather than authoritative.

#### `RULE-ALLOC-103` Flexible Money
Flexible Money is the explicit portion of actual Available Money the user has designated for discretionary spending. It is user intent, not a forecast and not identical to STS.

#### `RULE-ALLOC-104` Reserved Money
Reserved Money remains liquid but is protected for a purpose. It contributes zero to ordinary STS unless explicitly released/reallocated. A reservation linked to an obligation represents funding already protected for that obligation and MUST NOT cause a second deduction.

#### `RULE-ALLOC-105` Locked Money
Locked Money is intentionally unavailable for current spending. Technical withdrawability does not make it Flexible. It contributes zero to STS until explicitly unlocked/reallocated.

### 7. Income Allocation

#### `RULE-INCOME-ALLOC-001`
A realized income transaction first increases the receiving asset balance/Available Money. It MUST NOT be auto-classified as Flexible, Reserved, or Locked by a hardcoded income source rule. In the absence of an explicit allocation command, the entire newly available delta remains Unallocated.

#### `RULE-INCOME-ALLOC-002`
The product MUST allow the user to allocate newly received income to Flexible, Reserved, Locked, or leave it Unallocated. Previous choices MAY be suggested but do not execute automatically in V1.

### 8. Spending Budget Engine

A Budget is a cycle-scoped spending plan, separate from money allocation.

#### `RULE-BUD-101` Budget spending
For budget `b` and its Financial Cycle `c`:

```text
BudgetSpent(b,c)
= Σ recognized actual EXPENSE amounts
  assigned to categories mapped to b
  with effective date inside c
```

**Exclusions:** transfers, liability payments for purchases already recognized as expenses, income, future/expected spending, voided transactions, refunds to the extent they reverse eligible budget spending.

#### `RULE-BUD-102` Remaining and overspend

```text
BudgetRemaining = BudgetAmount - BudgetSpent
BudgetRemainingDisplayed = max(0, BudgetRemaining)
BudgetOverspend = max(0, BudgetSpent - BudgetAmount)
```

A negative raw remaining value is preserved semantically as overspend.

#### `RULE-BUD-103` Utilization
If `BudgetAmount > 0`:

```text
BudgetUtilizationPct = BudgetSpent / BudgetAmount × 100
```

If `BudgetAmount = 0` and `BudgetSpent = 0`, utilization is `0%`. If `BudgetAmount = 0` and `BudgetSpent > 0`, status is `OVERSPENT_UNBUDGETED`; percentage is not mathematically presented as finite.

#### `RULE-BUD-104` Warning status
Default V1 thresholds:

```text
0% <= utilization < 80%    = ON_TRACK_LIMIT
80% <= utilization < 100%  = APPROACHING_LIMIT
utilization = 100%          = LIMIT_REACHED
utilization > 100%          = OVERSPENT
```

Thresholds are constants/configuration-ready but not user-configurable in V1.

#### `RULE-BUD-105` Expense accounting interaction
An expense changes financial reality once through its transaction/account effect and changes budget consumption as a classification of that same expense. **Budget consumption MUST NOT subtract cash a second time.**

If an expense is paid from actual Flexible-funded money, Flexible Money is reduced by the realized spending amount, subject to explicit allocation reconciliation. Budget overspend therefore reflects that Flexible has already been consumed; it is not an additional deduction.

#### `RULE-BUD-106` Cycle close
Budgets do not financially roll over. At cycle close, the old Budget remains historical. Any unused budget amount does not create cash; the underlying unspent Flexible Money simply remains Flexible. The new cycle starts with no Budget instances unless the user creates/applies them.

#### `RULE-BUD-107` Previous-cycle template
“Use previous cycle as template” copies budget definitions/amounts/category mappings into **new Budget instances** for the new cycle. It does not move money and does not copy Spent.

#### `RULE-BUD-108` Budget is not reservation
Creating, increasing, decreasing, or deleting a Budget MUST NOT mutate Flexible, Reserved, Locked, Unallocated, Available Money, account balances, or STS by itself. A Budget is a spending plan only. Actual eligible expenses affect financial state once through Transaction/Account semantics and separately update Budget Spent.

#### `RULE-BUD-109` Budget capacity warning
For active Budgets in the active Financial Cycle:

```text
TotalActiveBudgetAmount = Σ BudgetAmount(b)
BudgetPlanOverFlexible = max(0, TotalActiveBudgetAmount - FlexibleMoney)
```

`TotalActiveBudgetAmount > FlexibleMoney` is valid. It MUST NOT block Budget creation/update solely for this reason and MUST NOT reserve money. When `BudgetPlanOverFlexible > 0`, surface warning state `BUDGET_EXCEEDS_FLEXIBLE` with the exact difference.

**Recalculate:** Budget create/edit/delete/cancel, Flexible allocation change, newly realized transaction that changes allocation state, allocation-shortfall resolution, and cycle transition.

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 10. Obligations and Applicable Upcoming Obligations

#### `RULE-OBL-101` Remaining due

```text
RemainingDue(o) = max(0, AmountDue - AmountSettled)
```

#### `RULE-OBL-102` STS applicability
An obligation is applicable to current STS when it is unsettled and its due instant is **on or before the end of the active Financial Cycle**, or it is already overdue. Future Expected Income never removes this requirement.

#### `RULE-OBL-103` Reserved coverage

```text
ApplicableReservedCoverage(o) = min(usable linked Reserved amount, ApplicableObligationAmount(o))
UnfundedObligationAmount(o) = max(0, ApplicableObligationAmount(o) - ApplicableReservedCoverage(o))
```

#### `RULE-OBL-104` Upcoming obligations

```text
UpcomingObligations = Σ RemainingDue(o)
```

for unsettled obligations inside the requested planning horizon. For STS use the active-cycle applicability rule and unfunded amount.

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 13. Budget Pace and Projection

#### `RULE-PACE-101` Cycle progress

```text
TotalCycleDays = count of local calendar dates in active cycle
ElapsedCycleDays = count from cycle start date through evaluation date, including today
CycleProgressPct = ElapsedCycleDays / TotalCycleDays × 100
```

#### `RULE-PACE-102` Budget consumption progress
For `BudgetAmount > 0`:

```text
BudgetConsumptionPct = BudgetSpent / BudgetAmount × 100
```

#### `RULE-PACE-103` Pace state
With tolerance `PACE_TOLERANCE = 5 percentage points`:

```text
BudgetConsumptionPct <= CycleProgressPct + 5pp  => ON_PACE
BudgetConsumptionPct >  CycleProgressPct + 5pp  => AHEAD_OF_PLAN
```

Budget limit statuses from `RULE-BUD-104` remain independently applicable.

#### `RULE-PROJ-BUD-101` Projected budget spending

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

#### `RULE-PROJ-BUD-102` Daily spending to return on track

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

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 15. Credit Card Engine

#### `RULE-CC-101` Purchase recognition
A credit-card purchase recognizes actual Expense at purchase time and increases Liability Outstanding by principal/recognized purchase amount. It does not reduce an asset account at purchase time.

#### `RULE-CC-102` Statement/cutoff
Purchases are assigned to statement periods using configured cutoff semantics. Statement amount due creates/updates an Obligation with configured due date.

#### `RULE-CC-103` Payment
Payment reduces the paying asset and Liability Outstanding and settles the obligation. It MUST NOT create a second category expense.

#### `RULE-CC-104` Installment conversion
Conversion supersedes the original future repayment representation and creates the actual installment schedule without duplicating purchase expense/principal.

### 16. BNPL / Installments / Fees

#### `RULE-INST-101` BNPL purchase
Purchase expense is recognized at purchase time. Liability includes purchase principal plus separately recognized fees/interest that the user is obligated to pay.

#### `RULE-INST-102` Schedule
Each installment has explicit due date and **actual required installment amount**. The engine MUST NOT assume `principal / count` when fees/interest/provider terms produce a different amount.

#### `RULE-INST-103` Fees/interest
Known purchase-time fees may be recognized as a separate fee expense and liability when incurred. Financing interest/fees that accrue later are recognized as expense/liability when contractually incurred according to the entered schedule. Payment itself is not a second expense.

### 17. Recurring and Staged Obligations

#### `RULE-REC-101` Recurring rule
A Recurring Obligation Rule generates obligation instances from recurrence and effective periods. Generated instances are canonical obligations for settlement/forecast/STS.

#### `RULE-REC-102` Staged amount
A rule MAY contain non-overlapping effective amount periods, e.g.:

```text
2026-10-01..2028-09-30  Rp3,200,000/month
2028-10-01..2030-09-30  Rp3,750,000/month
2030-10-01..open         Rp4,100,000/month
```

For an occurrence date, exactly one applicable stage MUST resolve. Overlapping stages are invalid.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 4. Money Allocation

#### `FR-ALLOC-001`
Show Flexible, Reserved, Locked, Unallocated and Available with conservation relationship.

#### `FR-ALLOC-002`
User explicitly changes allocation roles. Normal allocation commands cannot oversubscribe Available.

#### `FR-ALLOC-003`
If a truthful realized event causes allocation shortfall, retain the event, show exact shortfall, mark STS unresolved, and require explicit allocation resolution. Never silently choose which intent to reduce.

#### `FR-ALLOC-004`
Released Reserved/Locked defaults to Unallocated unless user chooses another destination.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 5. Budget

#### `FR-BUD-001` Create cycle budget
User creates a Budget with amount, name/purpose, Financial Cycle, and category mapping(s). Budget and Allocation remain separate. Budget creation MUST NOT move/reserve money. If Total Active Budget exceeds current Flexible Money, save remains allowed and the system MUST show `BUDGET_EXCEEDS_FLEXIBLE` with the difference.

#### `FR-BUD-002` Budget health
For every Budget show Budget Amount, Spent, Remaining or Overspend, utilization, status, and cycle days/progress.

#### `FR-BUD-003` Warning thresholds
Use default deterministic states: `<80% ON_TRACK_LIMIT`, `80..<100% APPROACHING_LIMIT`, `100% LIMIT_REACHED`, `>100% OVERSPENT`.

#### `FR-BUD-004` Spending pace
Show cycle progress versus budget consumption and `ON_PACE|AHEAD_OF_PLAN` per Financial Engine tolerance.

#### `FR-BUD-005` Projection
Budget utilization, cycle progress, and pace may be shown from Day 1. Projected Budget Spending and Projected Overspend MUST NOT be displayed until at least 3 Financial Cycle days have elapsed. Before then show projection unavailable/not yet eligible. From Day 3 onward, use the Financial Engine formula; if history is incomplete, show `INSUFFICIENT_DATA` rather than inventing a number.

#### `FR-BUD-006` Cycle close
Old Budget becomes historical. New cycle has no Budget instances until user creates/applies them. Account balances, liabilities, allocations, and persistent obligations do not reset.

#### `FR-BUD-007` Previous-cycle template
User can preview previous-cycle budget definitions, adjust them, then Apply to create new-cycle Budget instances with Spent=0.

#### `FR-BUD-008` Refund/category correction
Eligible refund or transaction recategorization MUST deterministically recompute Budget Spent for affected budgets/cycles.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 6. Obligations

#### `FR-OBL-001`
Create/edit/cancel/settle manual obligations with amount, due time, description/purpose, source and status.

#### `FR-OBL-002`
Partial settlement leaves Remaining Due active; overdue unpaid obligations remain visible and STS-applicable.

#### `FR-OBL-003`
Reserved money may link to an obligation and fund it without duplicate STS deduction. `Unfunded Obligation Amount = max(0, Applicable Obligation Amount - Applicable Reserved Coverage)`; STS deducts only this uncovered portion from Flexible.

#### `FR-REC-001` Recurring obligations
Create recurring rules for KPR, subscriptions, bills, recurring transfers/fixed payments.

#### `FR-REC-002` Staged amounts
Recurring rule supports effective amount periods. Overlap is rejected; correct stage generates each occurrence.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 10. Credit Card / BNPL / Installments

#### `FR-CRD-001`
Credit purchase recognizes Expense and increases Liability Outstanding at purchase time.

#### `FR-CRD-002`
Support statement/cutoff and due date; statement creates/updates repayment Obligation.

#### `FR-CRD-003`
Card payment reduces asset + liability and settles obligation; it MUST NOT create a second purchase-category expense.

#### `FR-INST-001`
Support conversion to installment and explicit installment schedule with actual installment amounts, due dates, fees/interest where known.

#### `FR-BNPL-001`
Support single-payment and installment PayLater. Purchase expense occurs at purchase; repayment is cash settlement, not duplicate purchase expense.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
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
| `EC-ALLOC-NEW-001` | Starting balance created during onboarding | Newly available amount is entirely Unallocated unless explicitly allocated; STS does not increase from Unallocated. | DERIVE |
| `EC-ALLOC-NEW-002` | Salary received | Actual balance/Available increase; new delta remains Unallocated until explicit allocation. | DERIVE |
| `EC-ALLOC-NEW-003` | Freelance income received | Actual balance/Available increase; new delta remains Unallocated until explicit allocation. | DERIVE |
| `EC-ALLOC-NEW-004` | User partially allocates newly available money | Allocated portion moves to selected role; remainder stays Unallocated. | ACCEPT |
| `EC-ALLOC-NEW-005` | User reallocates Flexible/Reserved/Locked | Apply explicit atomic allocation change; conservation must hold and no implicit destination is inferred. | ACCEPT |
| `EC-BUD-CAP-001` | Total Active Budget < Flexible | Valid; no budget-capacity warning. | DERIVE |
| `EC-BUD-CAP-002` | Total Active Budget = Flexible | Valid; no budget-capacity warning. | DERIVE |
| `EC-BUD-CAP-003` | Total Active Budget > Flexible | Valid; do not block; show `BUDGET_EXCEEDS_FLEXIBLE` by exact difference. No money is reserved. | WARN/CONFIRM |
| `EC-BUD-CAP-004` | Budget created/edited/deleted | Flexible/Reserved/Locked/Unallocated/Available do not change solely because of Budget mutation. | DERIVE |
| `EC-PROJ-004` | Budget created after cycle Day 3 | Projection is eligible immediately using cycle ElapsedCycleDays and all eligible mapped cycle spending. | DERIVE |
| `EC-PROJ-005` | Backdated eligible expense | Recompute Budget Spent, utilization, pace, projection/overspend if eligible, and Insights. | ACCEPT + SURFACE |
| `EC-PROJ-006` | Deleted/voided eligible expense | Remove its budget contribution and recompute projection/insights. | ACCEPT + SURFACE |

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

#### `EC-NUM-002` Budget projection

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

#### `EC-NUM-003` No financial rollover

```text
Old budget                      Rp1,500,000
Old spent                       Rp1,200,000
Unused budget                   Rp300,000
```

At cycle close, no Rp300,000 transaction is created. If the unspent cash was Flexible, it remains Flexible. A template can create a new Rp1,500,000 budget with Spent=0.


## Financial Calendar & Cycle Boundary Rules

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 9. Financial Cycle

#### `RULE-CYCLE-101` Configuration
A Financial Cycle has a recurring anchor day `D` and adjustment policy:

```text
NONE
PREVIOUS_WORKING_DAY
NEXT_WORKING_DAY
```

V1 working day means Monday–Friday in Financial Timezone. No public-holiday calendar is assumed.

#### `RULE-CYCLE-102` Nominal anchor
For each month, nominal anchor date is day `min(D, last_day_of_month)`.

#### `RULE-CYCLE-103` Adjustment
- `NONE`: effective anchor = nominal anchor.
- `PREVIOUS_WORKING_DAY`: if nominal anchor is Saturday/Sunday, move backward day-by-day to Friday; otherwise unchanged.
- `NEXT_WORKING_DAY`: if nominal anchor is Saturday/Sunday, move forward day-by-day to Monday; otherwise unchanged.

Do not hardcode “Friday” or “Monday” as rules; they are consequences of weekday evaluation.

#### `RULE-CYCLE-104` Boundaries
A cycle starts at local `00:00:00` on one effective anchor and ends immediately before the next effective anchor. Date presentation is inclusive: start date through calendar date immediately preceding next start.

#### `RULE-CYCLE-105` Active cycle
At evaluation instant `t`, active cycle is the unique cycle where `start <= t < next_start`.

#### `RULE-CYCLE-106` Anchor or adjustment-policy change
Financial Calendar is the sole authority for Financial Cycle boundaries. Changing the anchor or adjustment policy MUST NOT retroactively redefine the active cycle. Determine the transition from the boundary immediately following the active cycle to the first regular boundary under the new anchor and policy. If the transition duration is less than 15 days, it MUST NOT become a standalone cycle; merge it with the following regular cycle as an Extended Cycle. If it is 15 days or more, create a standalone Short Cycle before regular cycles under the new configuration.

> Source: [docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md). Source IDs and numeric examples are retained.

#### 4.2 `FinancialCalendarSettings` — Value Object

Conceptually contains:

```text
FinancialCalendarSettings
├── PaydayAnchor
└── AdjustmentPolicy
```

`PaydayAnchor` is valid only in `1..31`.

`AdjustmentPolicy` is one of:

- `NONE`
- `PREVIOUS_WORKING_DAY`
- `NEXT_WORKING_DAY`

Timezone is intentionally not embedded in this settings Value Object by the current Tactical decision. Timezone participates at the boundary that converts a current instant to a local financial date.

#### 4.3 `PendingConfigurationChange` — Value Object

A non-effective pending change has no independent domain identity. It can be replaced while it has not yet become temporally effective.

Conceptual fields:

```text
PendingConfigurationChange
├── RequestedSettings
├── RequestedOn        <<FinancialDate>>
├── OccurredAt         <<Instant>>
└── ProtectedThrough   <<FinancialDate>>
```

`RequestedOn` is the business financial date used to evaluate the effective configuration, resolve the currently protected Financial Cycle, and apply Financial Calendar rules. `OccurredAt` is the audit/event occurrence timestamp representing when the user/system operation occurred in absolute time.

`OccurredAt` MUST NOT implicitly determine a Financial Cycle boundary or business financial date. Any use of an `Instant` for Financial Calendar business semantics requires explicit conversion through the configured `FinancialTimezone` into a `FinancialDate` at the Application Service / Use Case boundary.

Audit history may retain superseded user actions, but superseded pending choices do not become effective domain revisions.

#### 4.4 `FinancialCycle` — Derived Value Object

`FinancialCycle` is an immutable derived domain value, not a persisted canonical historical Entity.

```text
FinancialCycle
├── StartDate
├── EndDate
└── CycleType
```

It has no independent `FinancialCycleId`. Value equality is based on its contained values.

`CycleType` is authoritative domain output from Financial Calendar and MUST NOT be independently inferred by consumers.

#### 4.5 `FinancialCyclePosition` — Derived Value Object

Cycle-relative position is separate from the immutable cycle because it depends on an `AsOfDate`.

```text
FinancialCyclePosition
├── AsOfDate
├── TotalCycleDays
├── ElapsedCycleDays
└── RemainingCycleDays
```

It is derived, never progressed as mutable daily state.

---

### 5. Temporal and Configuration Lifecycle

#### 5.1 Derived cycles, canonical configuration facts

Financial Cycles are resolved from configuration facts/history. Canonical state stores only the facts needed to deterministically reconstruct the calendar timeline; it does not pre-generate or persist every Financial Cycle as source of truth.

Core determinism property:

```text
same requested date
+ same effective configuration facts
= same FinancialCycle
```

#### 5.2 Protected active cycle

When configuration changes during the cycle containing the change date, that entire resolved cycle is protected and MUST NOT be retroactively redefined.

`ProtectedThrough` is the inclusive end date of that protected cycle.

#### 5.3 Effective date of a change

A requested configuration becomes temporally effective immediately after the protected cycle:

```text
EffectiveFrom = ProtectedThrough + 1 calendar day
```

Configuration effectiveness is not the same as the first regular payday boundary. The first cycle under the new configuration may be `SHORT` or `EXTENDED`.

#### 5.4 Temporal promotion requires no scheduler

Passage of time alone is sufficient to change the logical interpretation of a stored pending change. Financial Calendar correctness MUST NOT depend on a midnight job, background scheduler, or mutation that flips a persisted `PENDING` status to `ACTIVE`.

`PENDING` / `ACTIVE` are therefore not authoritative persisted lifecycle truth.

#### 5.5 Last pending configuration wins — only before effectiveness

A new requested configuration MAY replace an existing pending configuration only while the previous pending configuration has not yet become temporally effective.

Once temporally effective, the previous configuration is part of effective domain history. A subsequent user change is a new configuration change and protects the cycle that is active at that later change date.

#### 5.6 Materialization is bookkeeping

If persistence later normalizes a temporally effective pending representation into an effective revision representation, that normalization is not a new business fact and does not emit a Domain Event.

---

### 6. Initial Configuration and Historical Floor

Initial setup is not treated as a configuration transition because no prior Financial Calendar history exists to protect.

Given an initialization date, the initial settings may resolve the Financial Cycle containing that date even when the cycle starts before application/configuration creation.

Example:

```text
InitializedAt = 20 Sep
Anchor = 10

Initial cycle = 10 Sep .. 9 Oct
Initial revision EffectiveFrom = 10 Sep
```

`InitializedAt` and `EffectiveFrom` have different meanings:

- `InitializedAt`: when the calendar was actually configured;
- `EffectiveFrom`: earliest date from which the initial configuration is authoritative for cycle resolution.

The first revision's `EffectiveFrom` is the historical floor. Financial Calendar MUST NOT extrapolate the initial configuration backward before that date.

A request earlier than the historical floor produces `CalendarHistoryUnavailable`, not an invented cycle.

---

### 7. Regular Boundary Resolution

Regular boundary resolution follows this exact deterministic order:

```text
PaydayAnchor
    ↓
short-month clamp
    ↓
Nominal Boundary
    ↓
weekday AdjustmentPolicy
    ↓
Effective Regular Boundary
```

#### 7.1 Short-month clamping

For a year/month and `PaydayAnchor D`:

```text
nominalDay = min(D, lastDayOfMonth(year, month))
```

Examples:

- Anchor 31 in April → nominal 30 April;
- Anchor 31 in February → nominal 28/29 February;
- Anchor 30 in February → nominal 28/29 February;
- Anchor 29 in a non-leap February → nominal 28 February.

#### 7.2 Weekday adjustment

V1 working day means Monday–Friday only. Public holidays are not modeled.

- `NONE`: effective boundary = nominal boundary.
- `PREVIOUS_WORKING_DAY`: weekend nominal boundary moves backward to the preceding weekday.
- `NEXT_WORKING_DAY`: weekend nominal boundary moves forward to the following weekday.

An adjustment MAY cross a calendar-month boundary. The adjusted/effective boundary remains authoritative.

#### 7.3 Boundary monotonicity expectation

Consecutive regular effective boundaries must remain strictly ordered. This is an important candidate for property-based testing across years, anchors `1..31`, leap years, and all adjustment policies.

---

### 8. Adjustment Cycle Strategy

When a configuration change takes effect after `ProtectedThrough`:

1. Set transition start to the calendar day immediately after `ProtectedThrough`.
2. Resolve the first **adjusted/effective regular boundary** under the new settings.
3. Measure the transition duration in local financial dates from transition start through the date immediately preceding that regular boundary.
4. Classify the transition using the frozen threshold.

#### 8.1 Short Cycle

If transition duration is **15 days or more**:

```text
transition start .. day before first new regular boundary
= SHORT cycle
```

The following cycle begins at the first new regular boundary and is regular.

Exactly 15 days is `SHORT`.

#### 8.2 Extended Cycle

If transition duration is **less than 15 days**, the transition MUST NOT become an independent cycle. It is merged with the following regular cycle:

```text
transition start .. day before second new regular boundary
= EXTENDED cycle
```

#### 8.3 Adjustment before classification

Weekday adjustment is applied before calculating the `<15` / `>=15` transition classification. A weekend adjustment can therefore legitimately change whether the resulting cycle is `SHORT` or `EXTENDED`.

#### 8.4 Protected cycle may itself be non-regular

“Active cycle” means the cycle resolved for the change date, regardless of whether its type is `REGULAR`, `SHORT`, or `EXTENDED`. If the user changes configuration while in an `EXTENDED` or `SHORT` cycle, that entire cycle is protected before the next transition is calculated.

---

### 9. Local Date and Timezone Semantics

Financial Calendar's pure cycle model uses local financial calendar dates rather than UTC timestamps as the values of `StartDate`, `EndDate`, `EffectiveFrom`, `ProtectedThrough`, and `AsOfDate`.

`FinancialTimezone` is managed by the **Application Service / Use Case layer**, outside the Financial Calendar Aggregate and outside `FinancialCycleResolver`. The Application Service is responsible for reading the system clock, obtaining the configured `FinancialTimezone`, converting the current/evaluation instant into a local `FinancialDate`, and only then invoking the pure domain resolver:

```text
System Clock → Current Instant
                    +
          configured FinancialTimezone
                    ↓
              FinancialDate
                    ↓
          FinancialCycleResolver
          (pure domain service)
```

`FinancialCycleResolver` itself MUST NOT read the system clock, device timezone, own `FinancialTimezone`, or perform hidden UTC conversion.

`FinancialCycle.StartDate` and `FinancialCycle.EndDate` remain inclusive **Local Date** values. When the Financial Engine or another consumer requires an instant/timestamp range, the Application/Integration boundary MUST convert the cycle using the configured `FinancialTimezone` as a half-open range:

```text
InstantRange =
[start-of-day(StartDate, FinancialTimezone),
 start-of-day(EndDate + 1 day, FinancialTimezone))
```

Equivalently, the lower bound is inclusive and the upper bound is exclusive. UTC instants MUST NOT be stored inside `FinancialCycle` merely to support this integration representation.

Changing the configured timezone MUST NOT rewrite historical Financial Cycle date boundaries. It changes the derivation of the current local financial date and any required Local-Date-to-Instant conversion for subsequent time-sensitive operations.

Transaction timestamp/date reinterpretation after timezone changes is outside this bounded context and must be decided by Financial Accounts & Ledger.

---

### 10. Financial Cycle Position

For an inclusive cycle range and an `AsOfDate` inside that range:

```text
TotalCycleDays     = inclusive days(StartDate, EndDate)
ElapsedCycleDays   = inclusive days(StartDate, AsOfDate)
RemainingCycleDays = inclusive days(AsOfDate, EndDate)
```

Therefore:

```text
ElapsedCycleDays + RemainingCycleDays = TotalCycleDays + 1
```

The `+1` exists because `AsOfDate` belongs to both elapsed and remaining ranges.

On the final cycle day:

```text
RemainingCycleDays = 1
```

#### 10.1 Range invariant

`FinancialCyclePosition` may be calculated only when:

```text
Cycle.StartDate <= AsOfDate <= Cycle.EndDate
```

An out-of-range date MUST NOT be silently clamped. It produces an explicit domain rejection such as `AsOfDateOutsideFinancialCycle`.

This rejection is distinct from `CalendarHistoryUnavailable`.

---

> Source: [docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md). Source IDs and numeric examples are retained.

### 11. Commands and Business Outcomes

#### 11.1 Minimal V1 mutation command surface

```text
InitializeFinancialCalendar
ChangeFinancialCalendarConfiguration
```

The following are intentionally **not** business commands:

- `CreateFinancialCycle`
- `UpdateFinancialCycle`
- `DeleteFinancialCycle`
- `ActivatePendingConfiguration`
- `GenerateNextCycle`
- `CloseCurrentCycle`

Cycles and temporal effectiveness are derived.

#### 11.2 `InitializeFinancialCalendar`

Conceptual input:

```text
InitializeFinancialCalendar(
    settings,
    initializedOn
)
```

Behavior:

1. reject if already initialized;
2. validate/create settings Value Objects;
3. resolve the cycle containing `initializedOn`;
4. create initial `ConfigurationRevision`;
5. set first `EffectiveFrom` to that cycle's start date;
6. record `InitializedAt` separately.

#### 11.3 `ChangeFinancialCalendarConfiguration`

Conceptual input:

```text
ChangeFinancialCalendarConfiguration(
    requestedSettings,
    requestedOn
)
```

The command MUST evaluate logical effective/pending state as of `requestedOn`; it must not trust stale physical `ACTIVE/PENDING` labels.

It resolves the cycle containing `requestedOn`, protects that entire cycle, and establishes/replaces/cancels the pending configuration according to the rules above.

#### 11.4 Explicit business outcomes

Meaningful outcomes include:

- `Scheduled`
- `NoChange`
- `PendingChangeReplaced`
- `PendingChangeCancelled`

`NoChange` occurs when requested settings equal effective settings and there is no meaningful pending difference.

If a still-non-effective pending change exists and the user requests the currently effective settings, the pending change is cancelled rather than creating another change.

---

### 12. Queries and Output Contracts

#### 12.1 `ResolveFinancialCycle(requestedDate)`

Authoritative cycle query.

Conceptual result:

```text
CycleResolutionResult
├── Resolved(FinancialCycle)
└── CalendarHistoryUnavailable
```

A valid out-of-history request is not represented by ambiguous `null` and is not a technical exception.

#### 12.2 `GetEffectiveCalendarConfiguration(requestedDate)`

Returns a purpose-specific read contract such as:

```text
EffectiveCalendarConfiguration
├── PaydayAnchor
├── AdjustmentPolicy
└── EffectiveFrom
```

It does not expose a mutable Aggregate or require consumers to understand revision identity.

#### 12.3 `GetPendingCalendarConfiguration(asOfDate)`

Returns a pending change only if it is still logically pending as of the supplied date. A physically unnormalized record that is already temporally effective MUST NOT be reported as pending.

#### 12.4 `ResolveFinancialCycleContext(requestedDate)`

Financial Calendar MAY expose a coherent purpose-specific query returning:

```text
FinancialCycleContext
├── Cycle: FinancialCycle
└── Position: FinancialCyclePosition
```

Both values are resolved for the same requested date, reducing opportunities for consumers to combine mismatched temporal values.

#### 12.5 No hidden “current” primitive

Primitive domain queries accept explicit `requestedDate` / `asOfDate`.

A convenience use case such as `GetCurrentFinancialCycle` belongs outside the pure resolver and performs:

```text
Clock.now()
+ configured Financial Timezone
→ FinancialDate
→ ResolveFinancialCycle(date)
```

#### 12.6 No generic revision-history integration contract

Other bounded contexts MUST NOT receive a generic `GetAllConfigurationRevisions()` contract merely to reconstruct cycles themselves. A dedicated settings/history read model may be introduced only if an explicit UX/product requirement needs it.

---

### 13. Domain Services

#### 13.1 `FinancialCycleResolver`

Pure domain service responsible for deterministic cycle resolution.

Conceptual form:

```text
resolve(
    requestedDate,
    configurationTimeline
) -> CycleResolutionResult
```

Properties:

- no database access;
- no repository access;
- no system clock access;
- no hidden timezone/device-time lookup;
- no mutation;
- deterministic for the same inputs.

It is the sole place where Financial Calendar interprets effective configuration history and Adjustment Cycle Strategy to produce authoritative cycle boundaries/type.

#### 13.2 Cycle-position derivation

Cycle position is a pure derivation from `FinancialCycle + AsOfDate`. It may be implemented as behavior on a Value Object or as a small pure domain function/service; exact class placement is implementation detail so long as the ownership and invariants remain in Financial Calendar.

---

> Source: [docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md). Source IDs and numeric examples are retained.

### 15. Failure Semantics

Financial Calendar distinguishes at least three categories.

#### 15.1 Invalid input / Value Object construction failure

Examples:

- `PaydayAnchor = 0`
- `PaydayAnchor = 32`

Invalid values should be rejected at the input/Value Object boundary so invalid domain objects do not enter the Aggregate.

#### 15.2 Domain rejection

The request is technically well-formed but violates a domain invariant.

Examples:

- initializing an already initialized Financial Calendar;
- calculating `FinancialCyclePosition` with `AsOfDate` outside the supplied cycle.

Candidate explicit result: `AsOfDateOutsideFinancialCycle`.

#### 15.3 Valid domain/query outcome

The request is valid and the absence itself is meaningful.

Example:

```text
ResolveFinancialCycle(date before first revision EffectiveFrom)
→ CalendarHistoryUnavailable
```

This is not a technical failure and must not be silently converted to an invented cycle.

---

> Source: [docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md). Source IDs and numeric examples are retained.

### 18. Consolidated Invariants

1. Financial Calendar is the sole authority for Financial Cycle boundaries and cycle-relative temporal semantics.
2. Financial Cycles are derived values, not canonical persisted historical Entities.
3. The same authoritative configuration facts and requested date produce the same Financial Cycle.
4. A configuration change never retroactively changes the Financial Cycle containing the change date.
5. The protected cycle may be `REGULAR`, `SHORT`, or `EXTENDED`.
6. A new configuration is effective from `ProtectedThrough + 1 day`.
7. Temporal effectiveness requires no background mutation or scheduler.
8. Persisted `PENDING/ACTIVE` labels are not authoritative domain truth.
9. A pending change may be replaced only before it becomes temporally effective.
10. A temporally effective configuration becomes part of effective domain history; later changes are new changes.
11. Requesting identical effective settings with no meaningful pending difference is a domain no-op.
12. Requesting the effective settings while a non-effective pending change exists cancels that pending change.
13. Initial setup may back-resolve only the Financial Cycle containing initialization.
14. The first revision's `EffectiveFrom` is the start of that initial cycle and establishes the historical floor.
15. No Financial Cycle is invented before the historical floor.
16. Payday Anchor is valid only in `1..31`.
17. Missing anchor dates in short months clamp to the month's final calendar day.
18. Boundary resolution order is anchor → clamp → nominal boundary → weekday adjustment → effective boundary.
19. Working-day adjustment uses weekdays only in V1; no public-holiday calendar is modeled.
20. Weekday adjustment may move an effective boundary into an adjacent calendar month.
21. Short/Extended classification uses the first adjusted/effective regular boundary under the new configuration.
22. Transition `<15` days merges with the following regular cycle into an `EXTENDED` cycle.
23. Transition `>=15` days forms an independent `SHORT` cycle.
24. Exactly 15 transition days is `SHORT`.
25. `CycleType` is assigned by Financial Calendar and is part of authoritative `FinancialCycle` output.
26. `FinancialCycle` is immutable and has no independent identity.
27. Financial Calendar's pure cycle model uses local financial dates; hidden clock/device-timezone access is prohibited.
28. Timezone converts instants to financial dates outside the pure resolver.
29. Changing timezone does not rewrite historical cycle date boundaries.
30. `FinancialCyclePosition` is derived from a cycle and an in-range `AsOfDate`.
31. `TotalCycleDays`, `ElapsedCycleDays`, and `RemainingCycleDays` use inclusive date semantics.
32. On the final cycle date, `RemainingCycleDays = 1`.
33. `ElapsedCycleDays + RemainingCycleDays = TotalCycleDays + 1`.
34. Out-of-range cycle-position dates are rejected, never clamped.
35. Other bounded contexts consume authoritative cycle/position outputs instead of reimplementing their formulas.
36. Primitive Calendar queries use explicit dates; “current” is an application-level composition with Clock + configured timezone.
37. Query contracts do not expose mutable Aggregate internals or generic revision history for cross-context reconstruction.
38. Passage of time alone does not emit a Domain Event.
39. No activation event is required for financial correctness.
40. No-op commands emit no Financial Calendar Domain Event.
41. Persistence normalization/materialization is not a Domain Event.
42. Domain history, Domain Event history, and audit history are distinct and need not correspond one-to-one.
43. Repository operations do not invent business Domain Events.

---

### 19. Decision Register — TD-CAL-001 through TD-CAL-048

| ID | Accepted decision |
|---|---|
| TD-CAL-001 | Financial Cycles are deterministically resolved, not persisted as canonical state. |
| TD-CAL-002 | Store configuration/change facts needed to reconstruct the timeline; cycles remain derived. |
| TD-CAL-003 | Last Pending Configuration Wins while the pending configuration has not become effective. |
| TD-CAL-004 | New configuration is effective immediately after `ProtectedThrough`; `EffectiveFrom = ProtectedThrough + 1 day`. |
| TD-CAL-005 | Effective promotion is temporal/deterministic and requires no background job. |
| TD-CAL-006 | Pending replacement is allowed only before the pending configuration becomes temporally effective. |
| TD-CAL-007 | `PENDING/ACTIVE` are not authoritative persisted lifecycle states; effectiveness is derived from temporal facts. |
| TD-CAL-008 | `FinancialCycle` is an immutable derived Value Object without independent identity. |
| TD-CAL-009 | `CycleType` is authoritative Financial Calendar output and must not be independently inferred by consumers. |
| TD-CAL-010 | Adjusted/effective payday boundary is the authoritative cycle boundary. |
| TD-CAL-011 | Weekday adjustment occurs before Short/Extended transition classification. |
| TD-CAL-012 | Working-day adjustment may move an effective boundary into an adjacent calendar month. |
| TD-CAL-013 | Missing 29/30/31 anchors in short months clamp to the last calendar day. |
| TD-CAL-014 | Boundary resolution order is anchor → clamp → nominal boundary → weekday adjustment → effective boundary. |
| TD-CAL-015 | Financial Calendar models cycle boundaries using local calendar dates, not UTC timestamps, in its pure domain model. |
| TD-CAL-016 | Timezone converts current/evaluation instant to local financial date outside `FinancialCycleResolver`; resolver has no hidden clock/timezone dependency. |
| TD-CAL-017 | Timezone changes do not retroactively rewrite historical Financial Cycle date boundaries. |
| TD-CAL-018 | `FinancialCalendarConfiguration` is the Aggregate Root for configuration-timeline invariants. |
| TD-CAL-019 | Effective `ConfigurationRevision` is an Entity with identity and `EffectiveFrom`; settings are Value Objects. |
| TD-CAL-020 | `PendingConfigurationChange` is a Value Object without independent identity; audit history is separate. |
| TD-CAL-021 | Initial setup may resolve the cycle containing initialization even when its start precedes initialization; initial setup is not a transition. |
| TD-CAL-022 | Requesting settings identical to the effective configuration is a domain no-op when no meaningful pending difference exists. |
| TD-CAL-023 | Requesting the effective settings while a non-effective pending change exists cancels that pending change. |
| TD-CAL-024 | Calendar history begins at the start boundary of the cycle containing `InitializedAt`. |
| TD-CAL-025 | First revision `EffectiveFrom` equals the initial cycle start; `InitializedAt` separately records actual configuration time. |
| TD-CAL-026 | Resolver must not extrapolate before the first revision `EffectiveFrom`; return explicit unavailable/out-of-range result. |
| TD-CAL-027 | V1 mutation surface is limited to `InitializeFinancialCalendar` and `ChangeFinancialCalendarConfiguration`; cycle activation/generation/closing are not commands. |
| TD-CAL-028 | Configuration-change commands evaluate logical temporal state as of `requestedOn`, not stale persisted lifecycle labels. |
| TD-CAL-029 | Change operations distinguish `Scheduled`, `NoChange`, `PendingChangeReplaced`, and `PendingChangeCancelled`. |
| TD-CAL-030 | Distinguish valid domain outcomes, domain rejections, and malformed input/Value Object failures. |
| TD-CAL-031 | Domain Events represent meaningful mutations, not passage of time. |
| TD-CAL-032 | No activation/became-effective event is required for correctness. |
| TD-CAL-033 | Minimal V1 Calendar events distinguish initialization, pending scheduling, replacement, and cancellation. |
| TD-CAL-034 | `NoChange` produces no Financial Calendar Domain Event. |
| TD-CAL-035 | Materializing a temporally effective configuration into a normalized persistence representation is not a Domain Event. |
| TD-CAL-036 | Effective domain history, Domain Event history, and audit history are distinct. |
| TD-CAL-037 | Repository load/normalization/migration/cache/re-persistence must not invent Domain Events. |
| TD-CAL-038 | `ResolveFinancialCycle(requestedDate)` is the authoritative cycle query; consumers must not reconstruct cycles independently. |
| TD-CAL-039 | Resolution returns explicit `Resolved(FinancialCycle)` or a domain outcome such as `CalendarHistoryUnavailable`, not ambiguous `null`. |
| TD-CAL-040 | Primitive Calendar queries accept explicit `requestedDate`/`asOfDate`; current-date convenience use cases compose Clock + timezone outside the resolver. |
| TD-CAL-041 | Query contracts expose purpose-specific DTO/read models, not mutable Aggregate internals or generic revision history. |
| TD-CAL-042 | Financial Calendar owns authoritative `FinancialCyclePosition` derivation separate from `FinancialCycle`. |
| TD-CAL-043 | Cycle-day counts use inclusive semantics; final-day `RemainingCycleDays = 1`. |
| TD-CAL-044 | `FinancialCyclePosition` is derived, not authoritative mutable/persisted daily state. |
| TD-CAL-045 | Consumers use Financial Calendar's elapsed/remaining-day semantics rather than redefining them. |
| TD-CAL-046 | `FinancialCyclePosition.AsOfDate` must be within the supplied cycle; no silent clamping. |
| TD-CAL-047 | Out-of-cycle position calculation is explicit domain rejection and differs from `CalendarHistoryUnavailable`. |
| TD-CAL-048 | Financial Calendar may expose `ResolveFinancialCycleContext(requestedDate)` returning coherent Cycle + Position for the same date. |

---

> Source: [docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md). Source IDs and numeric examples are retained.

### 24. Candidate Test Matrix

These are audit-derived test targets, not new product requirements.

#### Boundary resolution

- Anchor `1..31` across all months.
- Leap and non-leap February.
- `NONE`, `PREVIOUS_WORKING_DAY`, `NEXT_WORKING_DAY`.
- Weekend adjustment crossing month/year boundaries.
- Property: consecutive effective regular boundaries are strictly increasing.

#### Transition strategy

- Transition 14 days → `EXTENDED`.
- Transition exactly 15 days → `SHORT`.
- Transition 16+ days → `SHORT`.
- Weekend adjustment changes nominal 15-day transition into adjusted `<15` transition.
- Configuration change while current cycle is `REGULAR`, `SHORT`, and `EXTENDED`.

#### Pending/effective lifecycle

- Replace pending before effectiveness.
- Cancel pending by returning to effective settings.
- App remains closed across `ProtectedThrough`, then reopens after effectiveness.
- New change after prior pending is temporally effective but physically unnormalized.
- No background scheduler available in any of the above.

#### Initial history

- Initialization in middle of cycle back-resolves to cycle start.
- Query on first `EffectiveFrom` succeeds.
- Query one day before historical floor → `CalendarHistoryUnavailable`.

#### Cycle position

- First day: `Elapsed = 1`.
- Last day: `Remaining = 1`.
- Identity `Elapsed + Remaining = Total + 1` for every date in cycle.
- Out-of-cycle `AsOfDate` → explicit rejection, no clamp.

#### Events

- Initialization emits initialization event.
- Schedule/replace/cancel emit distinct events.
- No-op emits no event.
- Passage of time emits no activation event.
- Persistence normalization emits no business event.

---

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 1. Onboarding & Settings

#### `FR-ONB-001` Short onboarding
Collect calculation currency/region as needed, Financial Timezone, cycle/payday anchor, weekday adjustment policy, and starting Cash balance. A default Cash asset account MAY be created. Other accounts MUST NOT be required to finish onboarding. Starting Cash MUST remain Unallocated unless the user explicitly allocates it during or after onboarding.

#### `FR-CYCLE-001` Configure cycle
User can set anchor day and `NONE|PREVIOUS_WORKING_DAY|NEXT_WORKING_DAY`. System previews effective next boundary and recalculates cycle-dependent outputs after changes.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-CYCLE-001` | Anchor 31 in shorter month | Nominal anchor = last calendar day, then apply adjustment policy. | DERIVE |
| `EC-CYCLE-002` | Nominal anchor Saturday, PREVIOUS | Walk backward to previous weekday. | DERIVE |
| `EC-CYCLE-003` | Nominal anchor Sunday, NEXT | Walk forward to next weekday. | DERIVE |
| `EC-CYCLE-004` | Adjusted next-month anchor crosses month boundary | Use resulting effective date; cycle remains previous effective anchor to next effective anchor with no overlap/gap. | DERIVE |
| `EC-CYCLE-005` | Public holiday on weekday | Treat as working day in V1 unless holiday mechanism is explicitly added later. | DERIVE |
| `EC-CYCLE-006` | New cycle starts | Account/liability/allocation state persists; prior Budgets close; new Budget Spent starts only in newly created instances. | DERIVE |
| `EC-CYCLE-007` | Anchor/policy changes during active cycle | Keep the active cycle unchanged. Financial Calendar resolves the transition beginning at its next boundary. | DERIVE |
| `EC-CYCLE-008` | New-anchor transition is under 15 days | Do not create a standalone cycle; merge the transition with the following regular cycle as an Extended Cycle. | DERIVE |
| `EC-CYCLE-009` | New-anchor transition is exactly 15 days or longer | Create a standalone Short Cycle before regular cycles under the new anchor. | DERIVE |


## Ledger Invariants, Corrections, Refunds & Ordering Rules

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 4. Account Balance and Liability Outstanding

#### `RULE-BAL-001` Asset balance
For asset account `a`:

```text
AssetBalance(a,t) = OpeningBalance(a) + Σ posted realized transaction legs affecting a after the opening snapshot and through t
  (for t at or after the opening snapshot)
```

Voided legs have zero current effect. The opening balance is a snapshot at an effective financial time. Pre-snapshot transactions are retained for history and eligible budget calculations but MUST NOT be applied again to the snapshot or subsequent balance. A balance before the opening snapshot is not inferred from the snapshot. When a date-only transaction and snapshot share a date and no times establish their ordering, obtain the user’s explicit before/after choice; include the transaction effect in the subsequent balance only when the user says after. Preserve the choice for deterministic recomputation. When a transaction correction crosses the snapshot boundary, re-evaluate its effect against the corrected position and recalculate affected eligible budget results; preview the impact and preserve audit. When editing the opening snapshot amount or effective date, preview current-balance impact and transactions whose relation changes, preserve the prior snapshot in audit, obtain confirmation, and recalculate affected results. Request the user’s before/after choice when applicable.

#### `RULE-BAL-002` Liability outstanding
For liability account `l`:

```text
LiabilityOutstanding(l,t)
= max(0,
    OpeningLiability(l)
  + recognized purchases/principal after the later opening through t
  + recognized fees/interest after the later opening through t
  - posted liability payments after the later opening through t
  - liability-reducing refunds/credits after the later opening through t)
  (for t at or after the later opening)
```

An earlier historical checkpoint is not added to the later opening. Its comparison with that opening follows `RULE-LDG-005`; equal timestamps that affect the result use the user ordering under `RULE-LDG-006`. The concrete ordering representation and stale-choice handling require tactical review. Do not infer a historical result when required facts or order are unavailable. Unused credit limit is excluded.

**Negative/zero:** floor at zero unless a separate positive asset/credit-balance representation is explicitly supported.  
**Recalculate:** purchase, fee/interest, refund, payment, void/edit, opening adjustment.

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 23. Ledger behavior clarifications — CR-LDG-001

These owner-approved product clarifications supplement the existing formulas; see [decision record](docs/archive/readiness/ledger-product-decisions.md).

#### `RULE-LDG-001` Negative tracked asset balance
A truthful asset-funded expense may produce a negative tracked asset balance. Display the negative balance and surface the need for review. `RULE-AVAILABLE-001` still includes `max(0, AssetBalance)` for each eligible account. Recompute allocation consistency; an allocation shortfall makes authoritative STS/Daily STS unresolved under `RULE-ALLOC-102` and `RULE-STS-103`. This does not create an overdraft facility.

#### `RULE-LDG-002` Payment above outstanding
Reject a new liability payment greater than recorded Liability Outstanding. For a backdated credit-card payment, validate the recorded debt at its financial time and the affected later history; if unsupported positive card credit results at any affected point, show the conflict and require correction without partial mutation. For date-only credit-card transactions sharing a date, when ordering affects payment validity obtain and preserve the user’s ordering choice instead of assuming an order. When recorded debt immediately before an old payment is unknown, surface it and require necessary starting information before validating/saving; never assume sufficient debt. No asset or liability mutation is committed on rejection. This V1 rule does not create a credit-balance asset; existing liability flooring is not a license to discard excess payment. A proposed backdated credit-purchase correction that would make an existing payment exceed recorded debt must be held for explicit conflict resolution without partially changing records or silently reducing the payment. A card refund greater than outstanding cannot be represented in full as a card credit in V1: explain the limitation and do not commit only the covered amount, discard the excess or record an unreceived bank refund.

#### `RULE-LDG-003` Adjustment, refund and fee classification
An explained balance adjustment changes actual account state without automatic ordinary income, expense or category classification. A linked refund reverses only attributable eligible expense/Budget Spent and must not become duplicate income. An optional transfer fee is a separately recognized fee expense; the transfer principal is not income, expense or Budget Spent. An unlinked refund increases the receiving account but is reported separately from ordinary income and expense and does not automatically change any category budget. Once explicitly linked after user confirmation, recompute only attributable eligible Budget Spent and retain audit history. Display balance adjustments separately in reports and use the corrected actual state as the forecast starting point. Transfer and fee corrections apply the user-confirmed effects atomically.

#### `RULE-LDG-004` Before calendar historical floor
Actual Ledger facts may predate the Financial Calendar historical floor. Calendar resolution before that floor remains `CalendarHistoryUnavailable`; no cycle may be inferred. Affected cycle-dependent budget results for the date are unavailable; affected cycle-based historical reports are marked incomplete or unavailable; show the transaction and account effect without inventing history.

#### `RULE-LDG-005` Earlier credit-card debt checkpoint
An earlier known card-debt amount at an effective time is a historical checkpoint. It is not added to the existing later opening liability; the later opening remains the starting point for that point and subsequent balances. For comparison, use the earlier checkpoint and applicable realized intervening effects under established financial-time ordering, and compare the resulting debt at the later opening with that recorded opening balance. If both values are determinable and differ, surface the difference and mark the affected historical calculation incomplete or unavailable until the discrepancy is resolved; never silently change a checkpoint or generate a transaction. If history or ordering is insufficient to compute the comparison, mark it indeterminate rather than reporting an invented numerical discrepancy. Existing `RULE-LDG-002` payment-time validation remains in force: unknown debt cannot validate an old payment. Storage representation, same-instant ordering, and exact comparison interval semantics require tactical review; these do not authorize double counting.

#### `RULE-LDG-006` Equal-time financial ordering
When realized financial facts have equal explicit timestamps and their ordering affects a result, require and preserve the user's explicit ordering; this includes a transaction tied with an asset opening snapshot or historical card checkpoint. Do not invent extra timestamp precision or infer order from IDs, insertion order or processing order. When the order cannot change the result, no user ordering is needed solely for the tie. If the user cannot determine a required order, return an indeterminate result; do not commit a mutation that requires a definite result. Revalidate the stored choice when affected facts change. Existing date-only ordering rules remain applicable. See CR-LDG-008.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

### 4. Money, time and state

#### 4.1 Money

Canonical money uses integer minor units, never floating point. V1 supports IDR only. Validate the transaction's positive amount and supported currency without inferring how balances or credit limits should be bounded. Rounding for division-derived outputs is governed by the engine; Ledger must not independently invent a rounding policy.

#### 4.2 Financial time versus audit time

A date-only transaction belongs to that local financial date in the configured financial timezone. A supplied timestamp is interpreted as its resolved instant in that timezone. Creation/update/audit timestamps serve a different purpose from the economic effective time. Backdating can affect historical budget assignment and downstream calculations; it must not silently rewrite the audit trail.

Financial Calendar resolves cycle membership for consumers requiring it. The frozen Calendar specification's historical floor and result semantics apply; do not assume the optional `ResolveFinancialCycleContext` query exists. An otherwise valid actual transaction may predate the floor; affected cycle-dependent budget results for that date are unavailable. Affected cycle-based historical reports must be marked incomplete or unavailable; do not invent a cycle.

#### 4.3 Current financial effect

For an asset account at or after its opening snapshot, the engine defines the snapshot balance plus posted realized legs after that snapshot through evaluation time; voided legs have zero current effect. Earlier transactions remain historical facts but do not change that snapshot or subsequent balance; pre-snapshot balances are not inferred. When a date-only transaction and opening snapshot share a date and their times do not establish order, ask the user to identify before or after; preserve the choice and apply its effect to subsequent balance only when after. Liability outstanding is the engine's floored-at-zero sum of opening liability, recognized purchases and fees, less payments and liability-reducing credits. New liability payments greater than recorded outstanding are rejected with guidance; no separate credit-balance asset is defined.

Balances may be computed or represented using rebuildable derived state, subject to the constitution. Their exact persistence and consistency mechanisms are open. Forecast inputs are assumptions and cannot create posted Ledger facts.

### 5. Transaction semantics

| Economic action | Required actual effect | Must not happen |
|---|---|---|
| Income | Increase receiving asset; newly available delta remains Unallocated by default. | Automatically link Expected Income or assign allocation based on source. |
| Asset-funded expense | Reduce asset; update eligible budget spending and dependent actual state. | Reject merely because truthful reality creates an allocation shortfall or deficit. |
| Owned asset transfer | Apply source and destination effects atomically. | Count the transfer as income, expense or budget spending. |
| Credit/PayLater purchase | Recognize expense at purchase and increase liability. | Decrease an asset merely for the purchase or treat credit limit as cash. |
| Liability payment | Reduce paying asset and liability; settle the relevant obligation as applicable. | Recognize the original purchase expense a second time. |
| Refund/credit | Apply the appropriate asset/liability effect and reverse attributable eligible spending. | Count a refund as duplicate income or reverse more budget spending than attributable. |
| Fee or interest | Recognize expense and liability when contractually incurred under the entered schedule. | Treat repayment as another fee expense or infer an unsupported installment amount. |
| Balance adjustment | Record an explained auditable change to actual account state, show it separately in reports, and start forecasts from corrected actual state. | Automatically classify it as income, expense or category spending. |

This table summarizes intent; source requirements govern detailed formulas. The split between supported operation variants and UI actions remains Q-06.

### 6. Account lifecycle and categories

#### 6.1 Account lifecycle

A user can create, edit and archive supported asset and liability accounts. Onboarding may create default Cash and must not require other accounts. Starting Cash is Unallocated until explicitly allocated. Financial eligibility and class are distinct from display or archive status. Archived eligible asset balances remain in Available under `RULE-AVAILABLE-001`.

A change to an account's financial data or eligibility can require recalculation of Available, allocations, STS, forecast and insights. The allowed mutations, their audit effect and treatment of existing transactions remain review items. The opening-balance correction mechanism must not create duplicate actual facts or silently rewrite historical effects.

#### 6.2 Category lifecycle

Categories classify relevant expenses and fees and can map to budgets. V1 must not depend on a founder-specific fixed taxonomy. Categories can be renamed or archived while preserving references; history shows the current name and retains rename audit. Archived categories are hidden from ordinary new entry. Merge and deletion remain open. A transaction recategorization must update affected budget spending for its effective cycle, while retaining auditability.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

### 8. Corrections and historical changes

A correction may change amount, account, category or effective time, or void an existing transaction. For each affected fact, identify the old and new financial effects and recompute dependent results from authoritative records. The system must preserve audit history and protect dependent relationships; `EC-REF-001` rejects destructive deletion of a record with dependent settlement/reconciliation in favor of the applicable void/cancel/supersede workflow.

| Change | Minimum effects to review |
|---|---|
| Void expense | Remove its current balance and eligible budget contribution; recalculate allocation/shortfall, STS, forecast and insights. |
| Recategorize | Remove old budget contribution and apply new eligible mapping for the relevant financial cycle. |
| Backdate or change date | Re-evaluate affected historical cycle, account state and budget/forecast dependencies. |
| Correct credit purchase across cutoff | Reassign statement/obligation atomically while preserving one purchase expense (`EC-CC-003`). |
| Correct payment with settlement | Reconcile liability and obligation settlement together; never leave an orphan or duplicate settlement. |
| Correct linked actual income | Coordinate actual transaction and expected-income association/lifecycle; apply the approved unlink timing rule or require guided resolution when intervening changes make it unsafe. |

Whether corrections use in-place revision plus audit, reversal and replacement, or another representation is not decided. The implementation is event-driven, not event-sourced. Effective-time changes and physical record edits are distinct from audit timestamps.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

### 12. Consolidated invariants for review

| ID | Candidate invariant restating baseline | Source |
|---|---|---|
| INV-LDG-01 | Each economic event is financially recognized once. | DM-PRINCIPLE-001; PRINCIPLE-004 |
| INV-LDG-02 | Actual transactions and forecast assumptions are distinct. | PRINCIPLE-001; RULE-FCST-101 |
| INV-LDG-03 | Canonical money uses integer minor units; a transaction amount is positive. | RULE-MONEY-001–002; DM-TRANSACTION-001 |
| INV-LDG-04 | Owned transfer effects commit together and do not count as spending or income. | FR-TRX-004; EC-TRX-001 |
| INV-LDG-05 | Purchase creates expense and liability; payment does not duplicate the purchase expense. | RULE-CC-101; RULE-CC-103 |
| INV-LDG-06 | Credit limit is never Available; eligible archived assets remain included. | RULE-AVAILABLE-001; FR-ACC-003 |
| INV-LDG-07 | Actual income may be unlinked; only explicit user intent links Expected Income. | FR-INC-003; FR-INC-005 |
| INV-LDG-08 | Actual newly available money remains Unallocated until explicitly assigned. | RULE-ALLOC-101A; FR-TRX-002 |
| INV-LDG-09 | A truthful expense may create a shortfall; it remains recorded and STS is unresolved. | FR-ALLOC-003; EC-TRX-003 |
| INV-LDG-10 | Corrections preserve audit and propagate all applicable consequences. | FR-TRX-005; NFR-DATA-003 |
| INV-LDG-11 | A failed financial mutation leaves no partial canonical change. | NFR-DATA-001; NFR-REL-001 |
| INV-LDG-12 | Cycle resolution belongs to Financial Calendar, not Ledger. | PRD-CYCLE-001; strategic DDD §5.6 |

The IDs above are draft review handles, not an approved decision register. Reconcile them with the source before freeze.

### 13. Worked examples

#### 13.1 Transfer

An asset account holds Rp1,000,000 and another holds Rp200,000. Transferring Rp300,000 produces balances Rp700,000 and Rp500,000. The combined balance is unchanged and no category budget is spent. If the destination effect cannot commit, the source effect cannot remain committed. Available may need recalculation when the accounts differ in liquidity eligibility; the example does not assume both are eligible.

#### 13.2 Purchase and payment

A credit purchase of Rp800,000 produces Rp800,000 expense and increases liability by Rp800,000, with no immediate asset reduction. A later Rp800,000 payment from an asset reduces that asset and the liability and settles the applicable obligation. It does not create another Rp800,000 purchase expense. Fees, if any, are separately recognized when incurred under the entered terms.

#### 13.3 Expected versus actual income

An expected salary of Rp5,000,000 is planning data. Posting actual Income of Rp4,800,000 increases the receiving asset by Rp4,800,000. Without a user-selected link, the occurrence is not marked received just because it resembles the actual Income. With a valid explicit link, Income Management applies the applicable lifecycle transition; no duplicate inflow is created. The newly available amount remains Unallocated until the user assigns it.

#### 13.4 Truthful shortfall

Available is Rp1,000,000 with Rp800,000 Flexible and Rp200,000 Reserved. A truthful Rp300,000 asset expense leaves Available at Rp700,000 while existing allocations total Rp1,000,000. The record remains and the Rp300,000 shortfall is surfaced; STS and Daily STS are unresolved until explicit resolution. The system does not silently select an allocation to reduce.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

#### 22.1 Suggested audit scenarios

1. Two refunds compete for the last Rp200,000 refundable amount; at most one valid set commits, and a failed link does not create a second refund inflow.
2. A transaction correction crosses the opening snapshot; compare before/after balances, budget effects, user-visible preview and audit, including a date-only same-day explicit choice.
3. A proposed backdated purchase correction invalidates an existing payment; preserve the old purchase, payment, bank balance and settlement until a valid set of changes commits atomically.
4. Explicitly linked salary is voided before and after expected time; verify EXPECTED/MISSED, no partial writes on unsafe intervening lifecycle change and no duplicate forecast event.
5. Archive an eligible nonzero account, rebuild Available and allocation state after restart, and verify it remains included while new entry requires restoration.
6. Transfer with a charged fee: simulate failure at each financial mutation, then confirm selected void/refund behavior and no partial update.

These scenarios are proposed checks, not completed tests. Explicit snapshot-date and historical liability edge cases must be completed before they can be treated as executable test oracles.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

#### 27.6 Review cases and disposition

| Candidate verification | Expected result |
|---|---|
| Two concurrent refund links each fit individually but exceed the purchase together | At most a valid combination commits; no duplicate inflow, silent link or lost refund. |
| Preview correction, then modify a dependency before confirmation | The stale preview is rejected and refreshed; no partial correction commits. |
| Rp700,000 earlier checkpoint plus complete intervening effects yields Rp950,000 against Rp1,000,000 opening | Show Rp50,000 discrepancy and incomplete/unavailable affected history; preserve both balances and create no transaction. |
| Same comparison with unknown intervening time/order | Show indeterminate, not a fabricated Rp50,000 discrepancy. |
| Earlier ordering choice becomes inapplicable after an edit | Ask again only if ordering now affects validity; preserve existing canonical facts if user cancels or validation fails. |
| Rebuild after a checkpoint correction or archived-account change | The rebuild agrees with the correctly committed canonical facts and visibly preserves unavailable or incomplete states. |

These are candidate tests, not test results. AUD-LDG-001–004 and AUD-LDG-008 still require contract review, boundary completion and verification. Same-instant checkpoint ordering, liability opening-state completeness, unsupported card-credit representation, exact applicability of ordering choices and baseline reconciliation remain open. Do not freeze this document on the strength of this proposal alone.

> Source: [docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md). Source IDs and numeric examples are retained.

### 29. Consolidated tactical model — review candidate

**Status:** FROZEN v1.0 by owner instruction, 2026-11-18, subject to open findings in §30. This section consolidates and supersedes provisional alternatives in §§3, 7, 9, 10, 19 and 27 where they overlap. Product and financial baseline remains authoritative. Audit findings in §22 record the state at the earlier review date; §30 records the current disposition. No SQL schema, endpoint, exact dispatch implementation, platform-specific framework API, completed tests or independent approval is implied.

#### 29.1 Responsibility and aggregate boundaries

The candidate uses three roots: **Account**, **Transaction**, and **Category**. Account owns class, lifecycle, eligibility, opening balance fact and account-specific terms. Transaction owns its kind, identity, status, economic time, amount and the complete set of account effects belonging to one economic event; transfer is one transaction with source and destination effects. Category owns its name and archive state, with historical references preserved. Audit revisions are durable records associated with a mutation, not an event-sourced system of record. Financial balances are derived from opening facts and valid posted effects.

The candidate selects an **application-level atomic consistency boundary**, not a single giant aggregate, for multi-root changes and financial dependencies. A transaction-local unit of work includes the authoritative affected Ledger facts, any required user ordering/link intent, and the owning contexts' critical canonical changes. Within that same commit it protects all refund links to the original purchase, enforces the maximum refundable amount and validates relevant payment/settlement constraints. Concurrent commands must serialize or detect a revision conflict and retry/re-preview; two valid individual reads may not both commit an invalid combined total. This is an invariant and integration contract, not a choice between optimistic locking and database serialization.

#### 29.2 Temporal and checkpoint evaluation

Each calculation takes an explicit evaluation time and the relevant canonical effective financial times, opening/checkpoint facts and applicable user ordering choices. For asset balances at/after the opening, include the opening once and only effects established as after it and no later than evaluation. For card balances at/after the later opening, include the later opening once and applicable later effects; an earlier card checkpoint is not added. For a card checkpoint comparison, apply effects established as after the earlier checkpoint and before the later opening; an event tied to either boundary belongs to the relevant side according to the approved user ordering when it affects the result. Void effects contribute zero. These are logical boundaries, not invented sub-second timestamps.

A comparison is `MATCHED` if complete inputs agree, `DISCREPANCY` with the determinable difference if complete inputs disagree, or `INDETERMINATE` if the necessary history or material ordering cannot be established. Distinguish an unknown as-of account balance from a known zero. A discrepancy never creates a transaction or changes either recorded balance. An older payment cannot be committed as valid without established applicable historical debt; if unsupported credit would arise at the payment or in affected later history, require explicit resolution without partial mutation.

For equal explicit timestamps or financially material same-day date-only ordering, retain an explicit user relationship to the relevant facts rather than synthesizing timestamps. A choice is reusable only while the same identified facts, effective-time relationship and meaning of the choice remain unchanged; any change that can alter those circumstances invalidates it and triggers revalidation. If no ordering can change the financial result, no user choice is required solely for ordering. If the user cannot determine a required ordering, return indeterminate and do not commit a use case requiring a definite outcome. Validate the logical ordering consistently; contradictory choices cannot become authoritative.

#### 29.3 Required mutation outcomes

| Use case | Required validation and authoritative effects | Non-success outcome |
|---|---|---|
| Account create/edit/archive/restore | Validate account class, supported money and dates; preserve class identity and archive history. Asset opening edits require preview, confirmation, audit and dependent recalculation. Archived eligible balances still contribute to Available. | Invalid input, unavailable history, stale preview or conflict leaves previously committed state unchanged. |
| Income with optional explicit expected occurrence | Record actual income once. If user selected an occurrence, coordinate Income Management's claim in the same critical operation. Newly available delta remains Unallocated. Unlinked actual income remains valid. | An explicitly requested claim that fails cannot leave half of the requested combined operation committed; user may submit a separate unlinked income action. |
| Expense and adjustment | Validate entered values and account use, post actual effect, and recalculate relevant budget and allocation status. An explained adjustment remains distinct from ordinary income/expense. Record truthful negative asset balance and show shortfall if applicable. | Invalid commands produce no mutation; financial shortfall resulting from truthful reality is an accepted surfaced outcome, not a reason to erase the event. |
| Transfer and optional fee | Require different source and destination. Commit both transfer effects and any selected fee expense together; fee is distinct from transfer principal. | Same-account or other invalid request rejects without partial effects. Correction previews user-selected transfer/fee results. |
| Payment and settlement | Validate debt at effective time and affected later history, necessary order and an applicable settlement with the Commitment owner. No unrepresented card credit or duplicate purchase expense. | Unknown debt/order, unsupported credit, stale approval or settlement conflict returns an explicit outcome and commits nothing for the requested operation. |
| Refund or association | Retain an unlinked actual refund as its own fact; a confirmed link updates eligible budget spending without creating an additional inflow. Check all valid links together against original purchase and supported liability representation. | Failed link of an existing refund preserves that existing fact; a newly requested create-and-link operation commits both or neither. |
| Correction or void | Read dependencies, preview, obtain required user choice, revalidate versions/order and apply all affected canonical facts and critical calculations together. Protect linked refunds, income claims, statements and settlements. | Invalidated ordering, dependency conflict or unknown time requires user resolution; preserve pre-operation state on cancellation/failure. |
| Historical card checkpoint | Preview checkpoint, comparison and affected transactions. Keep the later opening unchanged unless the user separately confirms an approved correction. Return MATCHED, DISCREPANCY, or INDETERMINATE; mismatched/unknown historical calculations are not represented as reconciled. | If payment validation requires unavailable historical debt, do not validate/save that payment. Creating the checkpoint itself must not invent a transaction or partial linked mutation. |

#### 29.4 Query and event integration contracts

Authoritative query families are account state with evaluation time and explicit available/unavailable status, transaction with current status and dependency references, eligible actual spending facts for a requested date range, historical checkpoint comparison with matched/discrepancy/indeterminate status, and auditable change history. Each consumer obtains cycle information from Financial Calendar rather than recalculating its boundaries. Query results involving more than one authoritative context must use a coherent committed view; inability to obtain it cannot be represented as a fresh authoritative amount.

Notifications report completed committed facts, including account and transaction mutations and association changes. They are in-process and may support noncritical reactions; they cannot substitute for any required critical transaction work. They must carry sufficient stable identity and provenance to identify the fact and recover secondary state, without including sensitive data in logs/analytics. Delivery, duplicate handling and handler-failure strategy belong in the later event catalog; a handler failure must not rewrite a committed fact. No event-sourcing, external broker or transactional outbox is introduced.

#### 29.5 Failure, audit and rebuild

Commands distinguish invalid input, domain conflict, missing or indeterminate history/order, unsupported positive card credit, stale preview/concurrent change and infrastructure failure. These are conceptual result categories, not public API names. A financially unfavorable but truthful expense is accepted and surfaced. Audit retains identity, relevant before/after facts, user ordering and link intent and originating operation without relying on replay as the current source of truth. Derived data can be rebuilt deterministically from canonical state and explicit effective time/order. Restore and migration must preserve the same invariants and must not claim unreconciled history is complete.

### 30. Review disposition and handoff

**Status:** The owner froze the consolidated baseline v1.0 on 2026-11-18 with the findings below still open. This is not independent approval, not evidence of implemented behavior, and not a release or implementation-readiness sign-off. This disposition supersedes the earlier dated audit table's wording about current freeze status while preserving that table as audit history.

| Finding | Current disposition | Remaining verification before freeze |
|---|---|---|
| AUD-LDG-001 | A cross-root atomic refund consistency contract is selected in §29.1 and §29.3. | Review and implement a concurrency test demonstrating that invalid combined links cannot commit. |
| AUD-LDG-002 | Owner-selected checkpoint, historical-payment and equal-time rules are mapped in §§28–29. | Review liability opening provenance/completeness and checkpoint interval edge cases against all baseline formulas; verify no floor conceals an invalid payment. |
| AUD-LDG-003 | §29.3 gives per-use-case all-or-nothing boundaries. | Reconcile concrete expected-income, commitment, statement and budget contracts when their tactical designs are written; verify coordinated rollback. |
| AUD-LDG-004 | Owner-selected boundary ordering and snapshot correction behavior is reflected in §29.2. | Verify exact before/after as-of query semantics and stale ordering-choice invalidation in review cases. |
| AUD-LDG-005 | Same-account transfer has a delegated recommended decision in CR-LDG-009; existing account/category rules are linked. | Review other operation-specific input rules and any remaining behavior not fixed by baseline; do not assume unlisted provider-specific behavior. |
| AUD-LDG-006 | Financial-engine wording now explicitly preserves unlinked actual income. | Review cross-document wording and an unlinked-income regression scenario. |
| AUD-LDG-007 | Unsupported card credit remains an explicit V1 limitation, not a silently represented balance. | Review incomplete-state communication and later actual bank-refund association; do not invent card credit. |
| AUD-LDG-008 | §29.4 specifies required read purposes and notification boundaries. | Complete concrete event/query catalogs during cross-context design; verify failure and rebuild semantics. |
| AUD-LDG-009 | Owner decisions and associated requirement, scenario and flow IDs are recorded. | Complete clause-level baseline and traceability audit; group-level mappings are not evidence of full coverage. |

Required decision scope for this candidate is the Ledger domain and contracts; implementation schemas, package names, exact transport types, synchronization and unrelated domain details may remain deferred within the architecture constitution. In contrast, explicitly unresolved user-visible or financially critical behavior cannot be silently deferred into implementation. The owner requested freezing with these findings open. Before implementing an affected path, complete its remaining review, reconcile actual source conflicts through change control, and record the responsible review and disposition. Proposed tests and author review must not be described as an independent audit or a passing test suite.

> Source: [docs/archive/readiness/ledger-product-decisions.md](docs/archive/readiness/ledger-product-decisions.md). Source IDs and numeric examples are retained.

## Ledger product decisions — change record CR-LDG-001

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. **Scope:** V1 Ledger product behavior below. **Technical design status:** Draft; acceptance does not freeze the tactical model or approve an ADR. This record does not claim independent review or completed implementation.

The functional requirements, engine, edge cases and user flows contain the corresponding clarifications under `FR-LDG-*`, `RULE-LDG-*`, `EC-LDG-*` and `UF-LDG-*`. Where a behavior was already frozen, this record clarifies it. Where it was previously unspecified, it records an owner-approved requirements change. No source changes Financial Calendar's historical floor. Requirements not affected by this record remain in force.

| ID | Owner-approved direction |
|---|---|
| D-LDG-01 | Archive hides the account from ordinary new transaction selection; restore before a new transaction. Preserve history and non-zero balance, including eligible Available contribution; historical corrections remain possible. |
| D-LDG-02 | Account class cannot be changed after creation. Create a correct account and use a guided correction process for affected history; do not reinterpret facts silently. |
| D-LDG-03 | Support an explained, auditable balance adjustment, displayed separately from ordinary income and expense and not automatically categorized as either. Recalculate actual state and allocation consistency. Show adjustments separately in reports and use the corrected actual balance as the forecast starting point. |
| D-LDG-04 | Allow edit and void with audit history and dependent recalculation; guide the user through affected dependencies. Existing restrictions on destructive deletion remain. |
| D-LDG-05 | Permit truthful asset-funded expenses that result in a negative tracked balance; surface it and suggest review. Eligible account contribution to Available remains floored at zero; apply existing allocation shortfall rules. This is not an overdraft or credit facility. |
| D-LDG-06 | Reject a new liability payment greater than recorded liability outstanding; explain the limitation and suggest checking for missing or incorrect transactions. A separate credit-balance representation is not supported by this decision. |
| D-LDG-07 | Default to a refund explicitly linked to its purchase and reverse attributable eligible spending. Allow a clearly labeled unlinked refund as a separate report item, neither ordinary income nor expense, without an automatic budget change. Later linking requires confirmation and recalculates the affected budget with audit history. |
| D-LDG-08 | Support category rename and archive without deleting historical references. Hide archived categories from ordinary new entry; display the current name in history and retain an audit of renames. Category merge and destructive deletion remain unspecified. |
| D-LDG-09 | Permit actual transactions before Calendar's historical floor, subject to valid transaction rules. Do not invent a cycle; mark affected cycle-dependent budget results unavailable for the date. Show the transaction and account effect; affected cycle-based historical reports are explicitly incomplete or unavailable, without inventing a cycle. |
| D-LDG-10 | Support a transfer with an optional separately recognized fee expense entered in the same flow. Transfer remains a transfer; fee is spending. Show linked transfer and fee together, preview edits and voids, and let the user choose each resulting change; commit the chosen changes together. On void, suggest voiding both while permitting the fee to remain if actually charged. |
| D-LDG-11 | When a linked actual income is edited or voided, show the linked occurrence and ask the user to confirm the appropriate correction; update dependent records consistently and atomically. After an unlink or void removes the supporting link, derive EXPECTED if the expected time has not passed and MISSED if it has; if intervening changes make the state unsafe to determine, require guided resolution and do not partially save. |
| D-LDG-12 | Recommend separate account and transaction concepts, atomic financial mutations, auditable corrections, conflict detection, rebuildable balances and purpose-specific reads. These are technical design direction, not approved aggregate boundaries or implementation contracts. |

### Remaining follow-up and review

- Q-04–06 remain open for aggregate design, cross-context atomicity and unchosen validation details. In particular, resolve actual-input rules for refunds and adjustments without inventing classification rules.
- Owner approved follow-up choices on 2026-11-14: refund reporting and later linking, adjustment reporting and forecast starting state, pre-floor report completeness, transfer-fee correction choices, and ordinary income unlink timing. Resolve concrete integration, validation and exceptional lifecycle contracts during tactical review; do not infer unsupported cancellation transitions.
- Update precise traceability and golden fixtures for these new identifiers, review the linked documents and obtain domain/architecture review. The product owner approved direction; no other approval is implied.

### Additional product decisions — CR-LDG-002

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Technical implementation and review remain outstanding.

| ID | Owner-approved direction |
|---|---|
| D-LDG-13 | Reject a refund link when the total valid linked refunds would exceed the original purchase. Explain the remaining linkable amount; never silently classify any excess. |
| D-LDG-14 | If correcting or voiding a purchase would make its linked refunds invalid, show the conflict and require explicit resolution before committing any affected change; do not silently void, reduce or unlink refunds. |
| D-LDG-15 | Correct an account created with the wrong class by creating the correct account and guiding explicit corrections of affected transactions, previewing effects and preserving audit/history; archive the old account when appropriate. Do not silently reinterpret all facts. |
| D-LDG-16 | V1 cannot represent a positive credit-card balance. If a refund to a fully paid card would require one, explain the limitation and do not silently discard money or record an unreceived bank refund. Guide the user to record the actual bank refund when it is returned, or correct erroneous history. If a card refund exceeds outstanding, do not record only the covered portion or discard the remainder; show that the full refund cannot be represented. |
| D-LDG-17 | An asset opening balance is a snapshot at an effective financial time. Transactions before it remain visible and may contribute to eligible historical budget activity, but are not applied again to the snapshot or subsequent asset balance. Do not infer account balance before the snapshot. When a date-only transaction and opening snapshot share a date and no times establish their ordering, ask the user whether the transaction occurred before or after the snapshot; apply it to subsequent balance only when after. |

This owner-approved product direction changes the prior unqualified asset balance formula; `RULE-BAL-001` is reconciled. The liability opening model has not been changed by this decision. Snapshot-day user choice is owner-approved; behavior when existing transactions or the snapshot are corrected across that boundary remains a review item, not an implied product decision.

### Correction decisions — CR-LDG-003

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. These decisions do not freeze the tactical model.

- **D-LDG-18:** When a corrected transaction moves across an asset opening snapshot, preview the effects; include or exclude its effect from subsequent account balance according to its corrected position, preserve history, and recalculate eligible budget results. Ask for before/after when ordering is unclear under `FR-LDG-012`.
- **D-LDG-19:** If a proposed backdated credit-purchase correction would make a recorded payment exceed recorded debt, surface the conflict and require explicit resolution before saving. Preserve the existing records unless a valid set of changes can be committed together; never silently reduce a real payment or discard its effect.

These choices address transaction correction across a snapshot and corrections invalidating an existing liability payment. The detailed technical mechanism, correction of the snapshot itself, and general historical outstanding behavior remain to be reviewed.

### Opening-snapshot and backdated payment decisions — CR-LDG-004

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. The Ledger tactical model remains a draft.

- **D-LDG-20:** An asset opening snapshot's amount or effective date may be corrected. Before saving, preview the impact on current balance and identify transactions whose relationship to the snapshot changes. Preserve the previous snapshot in audit history and recalculate affected results after confirmation. If a same-day transaction's order relative to the corrected snapshot is unclear, ask the user to select before or after under the existing snapshot rule.
- **D-LDG-21:** Validate a backdated credit-card payment against the recorded debt at the payment's financial time, and check the resulting affected later history. If the operation would create unsupported positive card credit at an affected point, show the conflict and require user correction; do not partially save or silently floor away the conflict. When historical debt or event ordering cannot be determined, surface uncertainty rather than inventing an order. Detailed resolution of ambiguous liability ordering remains to be specified before that case is implemented.

These are approved product behaviors, not a decision about SQL schema or concurrency mechanism. Liability opening-state semantics, same-time ordering and exceptional flows still require review.

### Historical credit-card history decisions — CR-LDG-005

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. The Ledger tactical model is not frozen.

- **D-LDG-22:** If date-only credit-card transactions share a date and their order affects whether a payment is valid, ask the user which occurred first. Preserve the user's ordering choice for deterministic recalculation; do not assume purchases always precede payments. Do not require a choice when the order cannot affect the financial result.
- **D-LDG-23:** If recorded debt immediately before an old payment cannot be determined, show that historical debt is unknown and guide the user to provide or correct the necessary starting information before validating/saving that payment. Do not assume sufficient debt or fabricate an authoritative result. This does not decide that liability opening state follows asset snapshot semantics.

The detailed starting-information workflow, representation and handling of changes invalidating an earlier ordering choice remain technical/domain design work. Product approval is not an independent review.

### Historical balance and ordering revalidation — CR-LDG-006

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Tactical design remains a draft.

- **D-LDG-24:** Provide a guided way to supply an earlier known credit-card debt balance and effective date when needed to validate older history. Identify the later transactions that will be checked and preview impact before saving; preserve prior information in audit history and do not count the same debt twice. The owner subsequently approved the checkpoint relationship and discrepancy behavior in CR-LDG-007.
- **D-LDG-25:** Revalidate affected credit-card history when a transaction changes. Retain an earlier ordering choice only where it still applies; when a new order is needed to determine payment validity, ask the user before saving. Surface conflicts and do not partially save changes.

These choices approve a guided capability and revalidation behavior; they do not approve a multi-snapshot calculation formula, reconciliation algorithm, automatic debt adjustment or liability opening-state semantics.

### Historical card checkpoint reconciliation — CR-LDG-007

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Ledger tactical and persistence design remain subject to review.

- **D-LDG-26:** An earlier known credit-card debt is a historical checkpoint for validating older activity, not an additional debt. Keep the existing later opening balance as the starting point for that point and subsequent balances; never add the earlier checkpoint to it. Compare the earlier balance and attributable intervening transactions against the later balance, using established ordering and valid financial facts.
- **D-LDG-27:** If the comparison does not match, show the discrepancy and guide the user to investigate missing or incorrect records. Do not silently change either balance or create an expense, fee, payment or other transaction. Mark the affected historical calculation incomplete or unavailable until resolved; do not treat that history as fully reconciled or use unknown debt as a valid basis for an old payment. Preserve audit and preview affected changes.

An incomplete or indeterminate comparison is not proof of a numerical mismatch; show uncertainty rather than inventing missing facts. Checkpoint storage, simultaneous edits, same-instant ordering and the exact interface remain technical review items. The approved product outcome does not authorize inferred transactions, alter the separate asset snapshot rules, or declare the tactical specification frozen.

### Equal-time ordering — CR-LDG-008

**Decision:** Product owner selected the recommendation and authorized recommended product decisions for end-of-work review. Recorded 2026-11-18 (Asia/Jakarta). The Ledger tactical design remains under review.

- **D-LDG-28:** When realized financial events have exactly equal explicit timestamps and their order affects a financial result, ask the user for their order and preserve the choice. Do not ask solely to order events when the order cannot change the result. Revalidate the choice if affected records change. If the user cannot establish the required order, show an indeterminate result and do not commit an operation requiring a definite answer; preserve prior canonical state.
- **D-LDG-29:** Apply the same rule when a transaction and an asset opening snapshot or historical card checkpoint share an exact effective timestamp and their relationship affects the result. Ask whether the transaction belongs before or after the balance; do not fabricate a more precise timestamp. Existing same-day date-only rules remain in force.

This does not require invented timestamps, an arbitrary ordering by database identity, or a separate multi-device synchronization design. Ordering choices must be checked for consistency and revalidated when their source records change.

### V1 transfer validation — CR-LDG-009

**Decision:** Product owner authorized the assistant to select recommended product decisions, then explicitly requested freezing the Ledger baseline on 2026-11-18 (Asia/Jakarta). This records owner approval of the selected direction; technical findings remain open.

- **D-LDG-30:** A transfer between the same account is invalid. Explain that the source and destination must be different and suggest an appropriate correction or expense instead if that reflects the real event. Reject the proposed transfer without partial canonical mutation. This does not prevent a truthful expense or an explicit balance adjustment through their own supported flows.

This removes an ambiguity flagged by AUD-LDG-005. It does not define additional account classes, fee behavior or provider-specific logic.

### Owner freeze record — 2026-11-18

The product owner explicitly requested freezing the current Ledger baseline and product decisions. See [Ledger freeze record](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md#32-owner-freeze-record). This does not close open findings or imply independent technical review or executable test results.


> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 2. Accounts

#### `FR-ACC-001` Asset accounts
Create/edit/archive generic cash, bank, e-wallet, savings or other supported assets with liquidity eligibility.

#### `FR-ACC-002` Liability accounts
Create/edit/archive generic credit card, PayLater or other liabilities. Credit limit is informational and never Available Money.

#### `FR-ACC-003` Archive/history
Archiving preserves economic history and non-zero balance/liability effects.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 3. Transaction Capture

#### `FR-TRX-001` Quick expense
Default path MUST support calculator-style amount input → Category → Account → Save. Amount, category, and account are primary fields. Merchant/notes/tags/custom time/attachments are secondary unless a specific rule requires them.

#### `FR-TRX-002` Income
Posted Income increases actual receiving asset balance and, by default, increases Unallocated by the same newly available amount while existing explicit allocations remain unchanged. The system MUST then allow explicit destination allocation to Flexible/Reserved/Locked or leave it Unallocated; it MUST NOT hardcode or silently execute allocation by income source. Previous choices may be suggestions only.

#### `FR-TRX-003` Expense
Posted asset-funded Expense reduces actual account balance, updates eligible Budget Spent, allocation consistency, STS, Forecast starting state, and Insights.

#### `FR-TRX-004` Transfer
Owned-account Transfer uses atomic source/destination legs and does not count as income, expense, or Budget Spent.

#### `FR-TRX-005` Edit/void/correction
Correction MUST propagate to balances, liabilities, budgets, STS, Forecast, obligations/settlements, and Insights as applicable. Audit history is preserved.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 13. Atomicity and Error Behavior

- Multi-record economic actions commit atomically.
- Financially unfavorable states are valid and surfaced.
- Validation failure produces no partial canonical mutation.
- Derived values are not directly editable.
- Analytics failure never blocks financial state mutation.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 14. Ledger product clarifications — CR-LDG-001

Owner-approved 2026-11-14; see [decision record](docs/archive/readiness/ledger-product-decisions.md) for remaining design questions and review status.

#### `FR-LDG-001` Archive and account class
Archived accounts are hidden from ordinary new transaction selection and must be restored before a new transaction. Historical correction remains possible. Account class cannot be changed after creation; provide a guided correction path rather than silently reinterpreting historical facts. Archive preserves all historical and non-zero economic effects.

#### `FR-LDG-002` Balance adjustment and negative tracked balance
Allow an explained auditable balance adjustment, distinct from ordinary income and expense; do not automatically classify it as either. Recalculate actual state and allocation consistency. Allow truthful asset-funded expenses even if the tracked balance becomes negative; display the negative balance and suggest review. Existing shortfall semantics apply. Show adjustments separately in reports and use the corrected actual balance as the starting point for future forecasts.

#### `FR-LDG-003` Correct and void
Allow users to edit or void transactions, preserve audit history and propagate dependent changes. Surface dependent relationships and obtain confirmation when a user-visible correction choice is needed; destructive deletion restrictions still apply. When editing or voiding linked actual income, show the linked expected occurrence and obtain confirmation of the appropriate correction; update dependent records consistently and atomically. When the supporting link is removed, the occurrence becomes EXPECTED if its expected time has not passed, otherwise MISSED. If intervening changes make the state unsafe to determine, show the conflict and require guided resolution; do not partially save. A correction retaining a valid explicit link does not itself unlink the occurrence.

#### `FR-LDG-004` Liability payment limit
Reject a new liability payment greater than recorded Liability Outstanding and explain that the user can review missing or incorrect transactions. Do not discard an overpayment amount or silently represent it as zero. This does not define a separate credit-balance asset.

#### `FR-LDG-005` Refunds
Default to an explicitly purchase-linked refund; reverse attributable eligible budget spending without duplicate income recognition. An unlinked refund is clearly labeled and increases the receiving account; report it separately from ordinary income and spending and do not automatically change a category budget. If the user later links it to the original purchase, obtain confirmation, preserve audit history and recalculate attributable eligible budget spending without double counting.

#### `FR-LDG-006` Category lifecycle
Allow category rename and archive, preserve historical references, hide archived categories from ordinary new entry, display the current category name in history and retain a rename audit. Merge and destructive deletion behavior remain unspecified.

#### `FR-LDG-007` Transactions before calendar history
Allow an otherwise valid actual transaction dated before Financial Calendar's historical floor. Do not invent a cycle. Mark affected cycle-dependent budget results unavailable for that date. Show the transaction and its account effect, and mark affected cycle-based historical reports incomplete or unavailable.

#### `FR-LDG-008` Transfer fee
Allow an optional fee to be entered with a transfer; record it as a separately recognized fee expense, while the transferred amount remains a transfer. Show linked transfer and fee together during correction; let the user review and explicitly choose changes to each and commit selected changes together. When voiding the transfer, suggest voiding both but allow the user to retain the fee if it was actually charged; preview the final effects.

#### `FR-LDG-009` Linked refund limits and corrections
Reject a proposed link if total valid linked refunds would exceed the original purchase; explain the remaining linkable amount. If editing or voiding a purchase would invalidate its existing linked refunds, show the conflict and require explicit resolution before any affected mutation commits. Never silently reduce, void or unlink a refund. See CR-LDG-002.

#### `FR-LDG-010` Guided account-class correction
A user correcting an incorrectly classified account creates the appropriate account and is guided through explicit corrections of affected transactions, with preview and retained audit/history. The original account may be archived when appropriate. Do not automatically reinterpret historical transactions. See CR-LDG-002.

#### `FR-LDG-011` Credit refund beyond supported liability
V1 does not represent a positive credit-card balance. If an actual card refund would require such a balance, explain that it cannot be represented as a card credit, and do not silently discard the amount or record an unreceived bank refund. Guide the user to record a bank refund if and when actually received, or correct erroneous history. When the refund exceeds outstanding, explain that the full refund cannot be represented; do not record only the covered part or silently discard the remainder.

#### `FR-LDG-012` Asset opening snapshot
An asset account's opening balance is a snapshot with an effective financial time. Preserve pre-snapshot transactions in history and eligible historical budget calculations, but do not apply them again to the opening snapshot or subsequent account balance. Do not infer account balances before the snapshot. If a date-only transaction shares the snapshot date and no times establish order, ask the user whether it occurred before or after the snapshot; include its effect in the subsequent balance only when after and preserve the choice. When a transaction correction moves it across the snapshot boundary, preview the impact; include or exclude its effect from subsequent balance according to its corrected position, preserve history, and recalculate applicable budget results. Ask the user when the approved before/after rule requires it. When editing the snapshot amount or effective date, preview the impact on current balance and identify transactions whose relationship to the snapshot changes; preserve the prior snapshot in audit history, obtain confirmation, and recalculate affected results. Ask for before/after when same-day ordering is unclear.

#### `FR-LDG-013` Correction invalidating a liability payment
If a proposed backdated credit-purchase correction would make an existing recorded payment exceed recorded debt, show the conflict and require explicit resolution before saving. Preserve existing records until a valid set of changes can commit together; never silently reduce the payment or discard its financial effect. See CR-LDG-003.

#### `FR-LDG-014` Backdated credit-card payment
Validate a backdated credit-card payment against recorded debt at its financial time and check the affected later history. If it would create unsupported positive card credit at an affected point, show the conflict and require user correction before saving; no partial mutation or silent flooring. If date-only credit-card transactions share a date and their order affects payment validity, ask the user which happened first and preserve the choice; do not ask when the order cannot affect the financial result. If the recorded debt immediately before an old payment cannot be determined, show it is unknown and guide the user to provide or correct necessary starting information before validating/saving the payment. Do not invent an order or assume sufficient debt. See CR-LDG-004 and CR-LDG-005.

#### `FR-LDG-015` Guided historical card debt
When older card history cannot be validated because debt is unknown, provide a guided way to supply an earlier known debt balance and effective date. Identify later transactions to be checked and preview the impact before saving. Preserve prior information in audit history and do not count debt twice. Treat the earlier balance as a historical checkpoint rather than additional debt, and retain the existing later opening balance as the starting point for that point and subsequent balances. Compare intervening recorded activity against the later balance. If the values differ, display the discrepancy, guide investigation and mark affected historical calculations incomplete or unavailable until resolved; never silently change a balance or create a transaction. An indeterminate comparison must be surfaced as uncertain, not a fabricated discrepancy. See CR-LDG-006 and CR-LDG-007.

#### `FR-LDG-016` Revalidate saved card transaction order
When a transaction changes, revalidate affected card history. Retain a previous user ordering choice only if it still applies; ask for a new choice when the new order is needed to determine payment validity. Surface conflicts; do not partially save. See CR-LDG-006.

#### `FR-LDG-017` Equal-time ordering
When realized financial facts have exactly equal explicit timestamps and their order affects a financial result, ask the user to determine the order and preserve that choice. Apply the same principle to a transaction at the exact effective time of an opening snapshot or historical card checkpoint. Do not ask solely for order when it cannot change the result. Revalidate the choice when affected facts change. If the user cannot determine the order, surface an indeterminate result and do not commit an operation requiring a definite answer; preserve prior canonical state. See CR-LDG-008.

#### `FR-LDG-018` Same-account transfer
A transfer requires distinct source and destination accounts. If they are the same, reject the proposed transfer without partial canonical mutation and explain that the user must choose different accounts or record the actual event through the applicable flow. See CR-LDG-009.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-TRX-001` | Owned asset transfer | No income/expense/Budget Spent. | DERIVE |
| `EC-TRX-002` | Expense creates Funding Deficit | Keep truthful Expense; surface deficit. | ACCEPT + SURFACE |
| `EC-TRX-003` | Expense causes allocations > Available | Keep Expense; create allocation shortfall; STS unresolved until explicit resolution. | ACCEPT + SURFACE |
| `EC-TRX-004` | Void expense | Reverse balance and Budget Spent effects; recalc STS/Forecast/Insights. | ACCEPT + SURFACE |
| `EC-REF-001` | Delete record with dependent settlement/reconciliation | Reject destructive deletion; use void/cancel/supersede. | REJECT |
| `EC-CONC-001` | Concurrent allocation edits oversubscribe | Serialize/reject one commit; never silently violate normal conservation. | REJECT |
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


## Forecasting & Advisor Rules

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 14. Forecast Engine

#### `RULE-FCST-101` Actual starting point
Forecast starts from a snapshot of actual financial state at evaluation time. Forecast events MUST NOT mutate canonical actual state.

#### `RULE-FCST-102` Base projection
The **Base Forecast** has deterministic inclusion semantics:

```text
BaseProjectedBalance(t)
= ActualStartingLiquidBalance
+ Σ EXPECTED_STABLE income with expected_time <= t
+ Σ other deterministic future cash inflows explicitly modeled by existing V1 rules
- Σ scheduled future cash outflows through t
```

`EXPECTED_UNCERTAIN` income is excluded from Base Forecast. Expected Income remains a projection assumption and never mutates actual state or STS.

#### `RULE-FCST-103` Scenario Forecast
A Scenario Forecast starts from Base Forecast and adds only the explicitly selected `EXPECTED_UNCERTAIN` events:

```text
ScenarioProjectedBalance(t, S)
= BaseProjectedBalance(t)
+ Σ expected amount(e)
  for selected EXPECTED_UNCERTAIN event e in scenario S
  where expected_time(e) <= t
```

If no uncertain events are selected, Scenario Forecast equals Base Forecast. Scenario inclusion MUST be visible/explainable and MUST NOT mutate actual state.

#### `RULE-FCST-103A` Expected Income state transition
Income Management owns Expected Income lifecycle; Financial Accounts & Ledger owns actual Income. An occurrence remains `EXPECTED` until cancelled, its expected time passes without an explicit link (`MISSED`), or the user explicitly links a posted actual Income transaction (`RECEIVED`). The link is optional, so actual Income may remain unlinked. Amount/date/description/source similarity, date arrival, or another heuristic MUST NOT create the link or mark the occurrence `RECEIVED`. For date-only expectations, "passed" means after the end of that local financial date. A late actual receipt MAY move `MISSED` to `RECEIVED` only through the same explicit user link; actual amount/time remains authoritative.

#### `RULE-FCST-104` Projected cycle-end balance
Evaluate `ProjectedBalance` immediately before the next Financial Cycle start. Actual/assumed components MUST be explainable.

#### `RULE-FCST-105` No double counting
Liability purchase spending and later liability payment are distinct expense-vs-cash-flow stages. Forecast cash outflow includes the scheduled payment; spending analytics/budgets recognize the purchase expense, not payment again.

#### `RULE-FCST-106` Funding Dependency / Boundary Risk
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

> Source: [docs/archive/requirements/04-financial-engine.md](docs/archive/requirements/04-financial-engine.md). Source IDs and numeric examples are retained.

### 18. Deterministic Advisor / Insights

#### `RULE-ADV-101` Inputs only
Advisor numeric statements MUST be generated only from Financial Engine derived values. It MUST NOT invent, estimate with an LLM, or silently alter financial numbers.

#### `RULE-ADV-102` Insight structure
A financial insight SHOULD map deterministic data into:

```text
WHAT HAPPENED → WHY IT MATTERS → LIKELY OUTCOME → IMPACT → POSSIBLE ACTION
```

#### `RULE-ADV-103` Budget pace insight
When a budget is `AHEAD_OF_PLAN`, the advisor MAY state cycle progress, budget consumption, Projected Budget Spending/Overspend, and `MaxAverageDailySpendToStayWithinBudget` when available.

#### `RULE-ADV-104` Funding Dependency insight
When `RULE-FCST-106` detects Funding Dependency, the Advisor MAY explain the future obligation, the actual-only funding shortfall before that obligation, the `EXPECTED_STABLE` income scheduled before it, and whether Base Forecast remains deficient after that income. This insight MUST NOT change current STS. If the supporting income becomes `MISSED` or moves after the obligation, the Advisor MUST stop describing the obligation as dependent on that income and surface the resulting projected funding risk/deficit.

#### `RULE-ADV-105` No autonomous action
Advisor suggestions do not move money, edit budgets, change income assumptions, or change obligations without explicit user action.

### 19. Recalculation Triggers

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

### 20. Explainability Contract

For each important derived value the engine MUST expose or be able to reconstruct:
- definition and evaluation time;
- input records/amounts;
- exclusions;
- formula/rule IDs;
- rounding;
- status/assumptions;
- recalculation provenance sufficient for debugging.

Forecast explanations MUST label actual starting state separately from assumed future events.

### 21. Acceptance Invariants

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

### 22. Final Closure Invariants

- `RULE-CLOSE-001`: newly available money remains Unallocated until an explicit allocation command changes its role.
- `RULE-CLOSE-002`: Unallocated contributes zero to STS.
- `RULE-CLOSE-003`: Budget creation/amount changes never reserve or move money; `TotalActiveBudgetAmount > FlexibleMoney` is valid with warning.
- `RULE-CLOSE-004`: STS considers only applicable unfunded obligations due in or overdue into the active Financial Cycle; post-cycle obligations do not reduce current STS.
- `RULE-CLOSE-005`: `EXPECTED_STABLE` enters Base Forecast; `EXPECTED_UNCERTAIN` enters only selected Scenario Forecasts; neither increases current STS.
- `RULE-CLOSE-006`: numeric Budget projection is unavailable before 3 elapsed cycle days.
- `RULE-CLOSE-007`: Funding Dependency is derived from Forecast primitives and never mutates Actual state.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 7. Expected Income

#### `FR-INC-001`
Create Expected Income with amount, expected time, recurrence/source, and simple certainty `EXPECTED_STABLE|EXPECTED_UNCERTAIN`.

#### `FR-INC-002`
Expected Income may appear in Planning/Forecast/scenarios but MUST NOT change actual balances, Flexible, STS or Daily STS.

#### `FR-INC-003`
Financial Accounts & Ledger owns the posted actual Income; Income Management owns Expected Income lifecycle. While recording actual Income, the user MAY explicitly link a relevant Expected Income occurrence. The link is optional, actual Income may remain unlinked, and an explicit link changes `EXPECTED` to `RECEIVED`. No amount/date/description/source similarity or fuzzy/heuristic mechanism may auto-link or mark it `RECEIVED` in V1.

#### `FR-INC-004`
`EXPECTED_STABLE` income MUST be included in Base Forecast. `EXPECTED_UNCERTAIN` income MUST be excluded from Base Forecast and included only in explicit Scenario Forecasts.

#### `FR-INC-005` Late/missed expected income
When the expected time passes without a linked actual receipt, the Expected Income becomes `MISSED`; it MUST NOT become Actual Income automatically. Forecast and Funding Dependency recalculate. A later actual receipt MAY change `MISSED` to `RECEIVED` only when the user explicitly links it; actual amount/time is authoritative.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 9. Forecast / Planning

#### `FR-FCST-001`
Planning shows chronological future obligations, recurring events, liability payments, Expected Income and projected positions.

#### `FR-FCST-002`
Forecast distinguishes actual starting state from assumptions and base vs uncertain-income scenario where applicable.

#### `FR-FCST-003`
Projected Cycle-End Balance must be explainable by included inflows/outflows and assumption set.

#### `FR-FCST-004` Funding Dependency
Planning/Advisor MUST detect and explain when a future obligation lacks sufficient actual-only projected liquid funding before its due time and therefore depends on `EXPECTED_STABLE` income scheduled before it. This condition MUST NOT modify current STS. If supporting Expected Income is missed/delayed beyond the obligation, surface projected funding risk/deficit instead of treating it as funded.

> Source: [docs/archive/requirements/05-functional-requirements.md](docs/archive/requirements/05-functional-requirements.md). Source IDs and numeric examples are retained.

### 11. Advisor / Insights

#### `FR-ADV-001`
Contextual insight may appear in Home, Budget, Planning; no primary Advisor tab required.

#### `FR-ADV-002`
Insight structure SHOULD cover what happened, why it matters, likely deterministic outcome, impact, and possible action.

#### `FR-ADV-003`
Every numeric statement/action amount MUST reference a Financial Engine derived value; no UI/AI independent arithmetic or invented estimate.

> Source: [docs/archive/requirements/06-edge-cases.md](docs/archive/requirements/06-edge-cases.md). Source IDs and numeric examples are retained.

### Edge-case rules

| ID | Scenario | Required behavior | Type |
|---|---|---|---|
| `EC-INC-001` | Expected salary before obligation | Salary contributes zero to current STS; may improve Forecast only. | DERIVE |
| `EC-INC-002` | Uncertain freelance income | Base forecast excludes it; scenario may include it and must label assumption. | DERIVE |
| `EC-INC-003` | Actual income arrives early/late/with a different amount and user explicitly links it | Posted actual Income is authoritative; mark the Expected Income `RECEIVED` without duplicate inflow. | ACCEPT + SURFACE |
| `EC-INC-004` | Expected income missed/cancelled | Remove future forecast event; current actual STS was never dependent on it. | DERIVE |
| `EC-INC-005` | Similar actual Income exists but user does not link it | Keep actual Income valid and unlinked; do not mark Expected Income `RECEIVED` from amount/date/description/source similarity. | DERIVE |
| `EC-INC-006` | Late actual receipt is explicitly linked to a `MISSED` occurrence | Transition `MISSED` to `RECEIVED`; actual amount/time is authoritative. | ACCEPT + SURFACE |
| `EC-ADV-001` | Advisor wants numeric corrective amount | Must use engine output; omit amount if required input/projection unavailable. | DERIVE |
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
