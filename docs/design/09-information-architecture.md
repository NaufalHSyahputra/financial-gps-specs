# V1 Information Architecture

**Document ID:** IA-V1  
**Status:** APPROVED / FROZEN — Normative

## 1. Primary Navigation

```text
Home | Transactions | Budget | Accounts | Planning
```

- **Home:** What is safe now and what needs attention?
- **Transactions:** What actually happened?
- **Budget:** Am I spending according to plan this Financial Cycle?
- **Accounts:** What money/debt do I actually have and how is owned money allocated?
- **Planning:** What is expected to happen next?

Advisor/Insights has no required primary tab; insights appear contextually in Home, Budget, Planning.

## 2. Home
Priority:
1. STS / Funding Deficit / unresolved state;
2. Daily STS + active Financial Cycle;
3. budget attention summary;
4. upcoming obligations;
5. contextual deterministic insights;
6. compact account/allocation summary.

“Why this amount?” MUST explain STS using actual state, current-cycle horizon, and explicitly state that Expected Income and Unallocated Money are excluded until receipt/allocation respectively. Home MUST distinguish Total/Available Money from STS so `Available > 0` with `STS = 0` is understandable.

## 3. Transactions
Default expense capture:

```text
Tap +
→ calculator-style amount
→ Category
→ Account
→ Save
```

Secondary fields remain optional for ordinary capture. Timeline distinguishes Income, Expense, Transfer, credit/BNPL purchase, liability payment, refund/correction. Transfers are not visually summarized as spending.

## 4. Budget
Budget screen owns:
- cycle selector/current cycle;
- each Budget amount, Spent, Remaining/Overspend, utilization and warning;
- cycle progress vs consumption;
- deterministic projected spending/overspend when available;
- contextual insight/action;
- “Use previous cycle as template” at new cycle.

Budget MUST NOT be presented as Reserved/Locked money allocation and creating a Budget MUST NOT visually imply that money moved. If Total Active Budget exceeds Flexible, show a non-blocking warning with the difference. Projection area MUST suppress numeric projected spending/overspend before Day 3 while still showing utilization/progress/pace.

## 5. Accounts
Separate Assets and Liabilities. Asset/account detail shows balance/activity. Liability detail shows outstanding liability, purchases, statement/repayment schedule and obligations. Allocation management shows Flexible/Reserved/Locked/Unallocated separately from Budgets. When Available contains Unallocated money, show the actual amount and an explicit path to Allocate Money; never imply the user has no money merely because STS is zero.

## 6. Planning
Contains Forecast, upcoming/overdue obligations, Expected Income, recurring/staged obligations, credit/BNPL/installment future payments, and scenario assumptions.

Forecast MUST visually distinguish:
- actual starting state;
- `EXPECTED_STABLE` income included in Base Forecast;
- `EXPECTED_UNCERTAIN` income only in explicitly selected Scenario Forecasts;
- future outflows;
- projected position.

## 7. Empty/Attention States
- no budgets in new cycle → offer create/template, never silently roll over;
- Flexible=0 with Unallocated>0 → explain explicit allocation;
- Funding Deficit → show causes, not auto-unlock money;
- allocation shortfall → STS unresolved + resolution path;
- uncertain Expected Income → show assumption badge/label;
- insufficient projection history or Cycle Day 1–2 → show no numeric projection;
- post-cycle Funding Dependency → show the obligation, unreceived supporting stable income, and projected risk without changing current STS;
- missed expected income → remove its future support and surface resulting forecast risk/deficit.

## 8. Terminology Invariants
“Balance”, “Available”, “Flexible”, “Budget Remaining”, and “Safe to Spend” are distinct. “Actual” and “Projected” are always distinguishable. Credit limit is never styled as owned cash.
