# V1 Scope

**Document ID:** SCOPE-V1  
**Status:** APPROVED / FROZEN — Normative Boundary

## 1. In Scope

- single-user personal financial profile;
- short onboarding with Financial Cycle/payday anchor, adjustment policy, currency/timezone, and starting Cash balance/default Cash account;
- asset/liability accounts: cash, bank, e-wallet, savings, credit card, PayLater and generic supported types;
- manual Income, Expense, Transfer, liability purchase/payment, refund/correction;
- calculator-first quick expense entry;
- category/purpose-based cycle Budgets;
- Flexible, Reserved, Locked, Unallocated money allocation;
- actual-only STS and Daily STS;
- expected stable/uncertain income for Forecast only until explicitly linked to actual Income and marked `RECEIVED`;
- obligations, recurring/staged obligations, credit statements, BNPL and installments;
- deterministic Forecast and budget run-rate projection;
- deterministic contextual Advisor/Insights;
- auditability and correction propagation.

## 2. Explicit V1 Boundaries

### `SCOPE-ACTUAL-001`
Future Expected Income contributes zero to current STS and actual money metrics until received.

- Newly available money remains Unallocated until explicitly allocated; Unallocated contributes zero to STS.
- Total Active Budget may exceed Flexible Money; creation remains valid and produces a warning rather than a funding reservation.
- STS horizon is strictly the active Financial Cycle. Post-cycle obligations are handled by Forecast/Advisor Funding Dependency detection.
- `EXPECTED_STABLE` is included in Base Forecast; `EXPECTED_UNCERTAIN` is scenario-only.
- Projected Budget Spending/Overspend require `MIN_PROJECTION_ELAPSED_DAYS = 3`.

### `SCOPE-BUD-001`
Budget is a first-class cycle-scoped spending plan and is not an Allocation.

### `SCOPE-CYCLE-001`
Financial Cycle supports configurable anchor day and `NONE|PREVIOUS_WORKING_DAY|NEXT_WORKING_DAY` weekday adjustment. No external holiday calendar is required. Financial Calendar alone resolves boundaries. An anchor/policy change never redefines the active cycle: a transition shorter than 15 days merges with the following regular cycle as an Extended Cycle, while a transition of at least 15 days forms a standalone Short Cycle.

### `SCOPE-ALLOC-001`
Flexible is explicitly user-controlled. Reserved/Locked are protected. Unallocated remains undecided.

### `SCOPE-FORECAST-001`
Forecast may include future income/outflows and scenarios but MUST label assumptions and MUST NOT mutate actual state.

### `SCOPE-ADV-001`
Advisor is deterministic/contextual, not a conversational LLM. Numeric recommendations must come from engine formulas.

## 3. Budget Lifecycle Boundary

Budgets belong to one Financial Cycle. They do not financially roll over. At a new cycle, no budget instances exist until created/applied. “Use previous cycle as template” creates new instances with zero Spent. Account balances, liabilities, allocations, and persistent obligations do not reset merely because the cycle changes.

## 4. Navigation Direction

Primary navigation remains approximately:

```text
Home | Transactions | Budget | Accounts | Planning
```

Advisor/Insights appears contextually inside Home, Budget, and Planning; it has no required primary tab.

## 5. Out of Scope

- crypto and investment portfolio valuation;
- bill splitting/shared ledgers;
- advanced accounting/general ledger/tax;
- automatic bank/open-finance sync unless separately added;
- multi-currency consolidation/FX engine;
- public-holiday calendar dependency for payday adjustment;
- user-configurable budget warning thresholds in V1;
- complex income-allocation automation builder;
- probabilistic/ML spending forecast;
- LLM chatbot/advisor;
- autonomous transfers, borrowing, or savings actions;
- provider-hardcoded financial engine rules.

## 6. Scope Guardrails

- Do not expand V1 to every possible financial persona.
- Do not simplify away Budget in favor of Allocation.
- Do not allow forecast assumptions to alter current STS.
- Do not count transfers as spending.
- Do not count credit/BNPL repayment as the original purchase expense again.
- Do not require full financial-life configuration during onboarding.
