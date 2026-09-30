# V1 Canonical Glossary

**Document ID:** GLO-V1  
**Status:** APPROVED / FROZEN — Normative Terminology

All monetary formulas use integer minor units. Terms in this document have one canonical meaning across the suite.

## Account
A tracked financial container classified as **Asset** or **Liability**.

## Asset
An owned economic resource. Only positive balances of liquidity-eligible Asset accounts participate in Available Money.

## Liability
An amount owed. Credit limits/unused borrowing capacity are not owned money and contribute zero Available Money.

## Total Balance
A display aggregate of selected account balances. Because assets and liabilities have different semantics, “Total Balance” MUST NOT be used as a synonym for Available Money or STS.

## Actual Financial State
Financial state derived only from events that have occurred: posted transactions, actual account/liability balances, existing allocations, and known obligations. Expected Income is not Actual Financial State.

## Forecast Financial State
A projected state starting from Actual Financial State and applying explicit future assumptions/events. It does not mutate Actual Financial State.

## Available Money
Actual positive owned liquid money eligible for allocation:

```text
AvailableMoney = Σ positive liquidity-eligible Asset balances
```

Expected Income and credit limits are excluded.

## Flexible Money
The explicit portion of Available Money the user has designated for discretionary spending. Flexible is user intent; it is not automatically equal to STS.

## Reserved Money
Actual liquid money protected for a designated purpose. It normally contributes zero to STS. A reservation may fund an Obligation without causing a second deduction.

## Locked Money / Locked Savings
Actual owned money intentionally excluded from current spending. Technical withdrawability does not make it safe to spend.

## Unallocated Money

```text
Unallocated = Available - Flexible - Reserved - Locked
```

Money whose role has not yet been explicitly assigned. Newly available money—including starting balance and received income—remains Unallocated until an explicit allocation command changes its role. It contributes zero to STS.

## Spendable Money
For V1 documentation, use **Flexible Money** when referring to user-designated discretionary money and **Safe to Spend** when referring to engine safety guidance. “Spendable Money” SHOULD NOT be used as a separate derived metric because it creates ambiguity.

## Money Allocation
Classification of actual owned money by role: Flexible, Reserved, Locked, or derived Unallocated. Allocation answers: **“What role does this money currently have?”**

## Category
A spending purpose used to classify actual expenses and associate them with Budgets.

## Budget
A Financial-Cycle-scoped spending plan for one purpose/category set. Budget answers: **“How much do I plan to spend for this purpose in this cycle?”** It is not an Allocation or reservation. Creating/changing a Budget moves no money. Total Active Budget may exceed Flexible Money; this is valid and produces a non-blocking warning.

## Budget Amount
The planned spending limit for one Budget instance in one Financial Cycle.

## Budget Spent
Sum of actual eligible expense transactions mapped to the Budget during its Financial Cycle. Transfers and liability repayments already represented by purchase expenses are excluded.

## Budget Remaining

```text
BudgetRemaining = BudgetAmount - BudgetSpent
```

Negative value means overspending; display may separately show `BudgetOverspend=max(0,-BudgetRemaining)`.

## Budget Utilization
For positive Budget Amount:

```text
BudgetUtilizationPct = BudgetSpent / BudgetAmount × 100
```

## Budget Spending Pace
Comparison of Budget Utilization against elapsed Financial Cycle percentage. V1 considers consumption more than 5 percentage points ahead of cycle progress `AHEAD_OF_PLAN`.

## Projected Budget Spending
Deterministic run-rate projection:

```text
ProjectedBudgetSpending
= (BudgetSpent / ElapsedCycleDays) × TotalCycleDays
```

rounded down to minor units. It is not an AI prediction. The numeric projection is not eligible on Cycle Day 1 or Day 2; it becomes eligible when `ElapsedCycleDays >= 3` and required history is sufficiently complete.

## Financial Cycle
Canonical budgeting/safety period defined by recurring anchor day and adjustment policy. A payday-based cycle is one configuration; calendar month is not assumed.

Adjustment policies: `NONE`, `PREVIOUS_WORKING_DAY`, `NEXT_WORKING_DAY`. V1 working day means Monday–Friday; public holidays are not modeled.

Financial Calendar is the sole boundary authority. Configuration changes do not redefine the active cycle. A post-active-cycle transition shorter than 15 days merges with the following regular cycle as an **Extended Cycle**; one lasting at least 15 days is a standalone **Short Cycle**.

## Obligation
A known required outflow with amount due, due date, settlement state, and source. Examples: KPR payment, bill, credit-card statement, BNPL payment, installment.

## Recurring Obligation
An obligation generated from a recurrence rule. Its amount may vary by effective period/stage.

## Expected Income
A future income assumption owned by Income Management. Canonical certainty is `EXPECTED_STABLE` or `EXPECTED_UNCERTAIN`; lifecycle states are `EXPECTED|RECEIVED|MISSED|CANCELLED`. `EXPECTED_STABLE` is included in Base Forecast; `EXPECTED_UNCERTAIN` enters only selected Scenario Forecasts. Expected Income contributes zero to actual-money and STS metrics. It becomes `RECEIVED` only through an optional explicit user link to an actual Income transaction owned by Financial Accounts & Ledger. An actual Income may remain unlinked; date arrival or amount/date/description/source similarity cannot mark it received. A late explicit link may move `MISSED` to `RECEIVED`.

## Actual Income
A posted INCOME transaction that has actually occurred. It changes actual account balance/Available Money. The newly available amount remains Unallocated until explicitly allocated by the user.

## Applicable Reserved Coverage
The usable Reserved Money explicitly protecting an applicable obligation, capped at that obligation's applicable amount.

## Unfunded Obligation Amount
`max(0, Applicable Obligation Amount - Applicable Reserved Coverage)`. STS deducts only this uncovered portion from Flexible, so funded Reserved coverage is not subtracted twice.

## Safe to Spend (STS)
Current actual Flexible Money remaining after protecting applicable unfunded obligations through the end of the active Financial Cycle:

```text
RawSTS = FlexibleMoney - ApplicableUnfundedObligations
DisplayedSTS = max(0, RawSTS)
FundingDeficit = max(0, -RawSTS)
```

Expected Income contributes zero before an actual receipt, regardless of its lifecycle label.

## Funding Deficit
Amount by which applicable current-cycle obligations exceed actual Flexible funding after reservation coverage.

## Daily Safe to Spend

```text
DailySTS = floor(DisplayedSTS / RemainingCycleDays)
```

RemainingCycleDays includes today through the last date of the active cycle. If STS <= 0, Daily STS = 0. If STS is unresolved due to allocation shortfall, Daily STS is unresolved.

## Forecast
A deterministic projection starting from actual current financial state and applying explicit future cash-flow assumptions. Forecast must distinguish actual facts from assumptions.

## Projected Balance
Projected liquid balance at a future instant under a specified Forecast assumption set.

## Projected Cycle-End Balance
Projected Balance immediately before the next Financial Cycle begins.

## Liability Outstanding
Actual amount currently owed on a liability after recognized purchases/principal/fees and actual payments/refunds.

## Advisor / Insight
A deterministic interpretation of engine outputs explaining what happened, why it matters, likely projected impact, and possible corrective action. It is not an LLM chatbot and cannot invent numeric values.

## Transfer
Movement between owned accounts. It changes account locations/balances but does not count as income or expense.

## Base Forecast
Derived forecast that starts from actual liquid state, includes `EXPECTED_STABLE` future income and scheduled future outflows, and excludes `EXPECTED_UNCERTAIN` income. It never mutates actual state.

## Scenario Forecast
A Base Forecast plus explicitly selected `EXPECTED_UNCERTAIN` events. It is derived and never changes STS or actual balances.

## Funding Dependency / Boundary Risk
A Forecast/Advisor condition where a future obligation lacks sufficient actual-only projected liquid funding before its due time and therefore depends on unreceived `EXPECTED_STABLE` income scheduled before it. It does not extend or modify the current-cycle STS horizon.

## Budget Plan Over Flexible
`max(0, TotalActiveBudgetAmount - FlexibleMoney)`. A positive value produces a non-blocking warning; it is not an allocation shortfall and does not reserve money.

## Projection Eligibility
Numeric Projected Budget Spending/Overspend is eligible only when at least 3 Financial Cycle days have elapsed and required transaction history is sufficiently complete.

## Canonical Terminology Rules
- `GLO-RULE-001`: Budget and Allocation are never synonyms.
- `GLO-RULE-002`: Flexible Money and STS are never synonyms.
- `GLO-RULE-003`: Expected Income and Actual Income are never synonyms; only an explicit user link marks the Expected Income `RECEIVED`.
- `GLO-RULE-004`: Forecast values are never presented as actual balances.
- `GLO-RULE-005`: Liability payment is not a duplicate expense.
- `GLO-RULE-006`: “Financial Cycle” is canonical; “payday cycle” describes one configuration.
