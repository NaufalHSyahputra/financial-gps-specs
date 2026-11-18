# Product Vision — V1 Personal Finance Management

**Document ID:** PV-V1  
**Status:** APPROVED / FROZEN — Normative  
**Product Thesis:** **Financial GPS, not just an expense tracker.**

## 1. Problem Statement

Traditional expense trackers primarily answer **“Where did my money go?”** Users also need to know:
- How much money do I really have available now?
- Am I spending according to my budget?
- How much can I safely spend until the end of my current Financial Cycle?
- If I continue spending like this, what will my financial position look like?

A raw balance is insufficient because money can be reserved, locked, needed for obligations, or held alongside liabilities. V1 closes the gap between **recording financial history** and **making an informed next decision**.

## 2. Core Positioning

> **Financial GPS, not just an expense tracker.**

```text
PAST                         NOW                         FUTURE
Transactions                 Actual financial state      Expected income
Spending history             Budget health               Upcoming obligations
Category analysis            Flexible Money              Liability payments
                              Safe to Spend               Spending projection
                                                          Financial forecast
```

Expense tracking and budgeting are foundational inputs. Safe-to-Spend and Forecast are core capabilities, not secondary analytics.

## 3. Target User Philosophy

**The founder is User #1, not the only user the system may support.** V1 targets a focused persona: a salaried individual with recurring primary income, possible variable income, multiple spending methods, personal budgets, recurring obligations, and potentially credit/PayLater liabilities.

Use **configuration over personal hardcoding**. V1 MUST NOT hardcode a specific payday, bank, provider, category set, or rule such as “freelance income goes to car savings.”

## 4. Personas and JTBD

### `PERSONA-001` Salaried Multi-Account Planner
Uses cash/bank/e-wallet/credit, has recurring obligations and cycle budgets.

**JTBD:** When I make day-to-day spending decisions, tell me what is actually safe now and whether my spending pace is sustainable through my Financial Cycle.

### `PERSONA-002` Salary + Variable Income User
Has predictable salary plus uncertain freelance/bonus income.

**JTBD:** Let me plan with expected income without treating money I have not received as spendable.

### `PERSONA-003` Liability and Obligation User
Uses credit cards, BNPL/installments, KPR/subscriptions or staged recurring payments.

**JTBD:** Show spending when it happens, debt separately, and future cash obligations without double counting.

## 5. Core Value Propositions

- `VP-001` **Past-to-future visibility:** transactions, budgets, actual state, and forecast in one coherent model.
- `VP-002` **Actual-safe guidance:** STS uses actual financial state only.
- `VP-003` **Budget awareness:** users see limit, remaining, overspend, cycle progress, pace, and deterministic projection.
- `VP-004` **Explicit money intent:** newly available money remains Unallocated until the user explicitly assigns Flexible, Reserved, or Locked intent; Unallocated does not increase STS.
- `VP-005` **Explainable forecast:** `EXPECTED_STABLE` income is included in Base Forecast; `EXPECTED_UNCERTAIN` income appears only in selected Scenario Forecasts. Neither changes current STS.
- `VP-005A` **Boundary-risk visibility:** post-cycle obligations do not reduce current-cycle STS, but Forecast/Advisor identifies when future obligations depend on Expected Income that has not yet been received.
- `VP-006` **Deterministic insights:** advisor numbers come from the engine, not invented narrative.

## 6. Product Principles

- `PRINCIPLE-001` Actual and Forecast MUST remain strictly separated.
- `PRINCIPLE-002` Expected Income MUST NOT increase current STS before receipt.
- `PRINCIPLE-003` Budget and Allocation are separate concepts.
- `PRINCIPLE-004` The same economic event MUST NOT be financially counted twice.
- `PRINCIPLE-005` User intent is explicit; newly available money remains Unallocated and the system does not silently allocate, unlock, or reallocate money.
- `PRINCIPLE-005A` Budget is a spending plan only. Creating or increasing a Budget does not reserve, allocate, or deduct money, and Total Active Budget may exceed Flexible Money with a warning.
- `PRINCIPLE-006` Financial Cycle is canonical; calendar month is not assumed.
- `PRINCIPLE-007` Every important derived number must be explainable and recomputable.
- `PRINCIPLE-008` Record financial reality even when it produces an unfavorable state.
- `PRINCIPLE-009` Configuration over founder-specific hardcoding.
- `PRINCIPLE-010` Manual capture must be low friction enough to sustain accurate data.

## 7. V1 Success Criteria

V1 succeeds when a configured user can:
1. record common expenses quickly and correctly;
2. understand actual account/liability state;
3. create cycle-scoped category budgets and see budget health;
4. see deterministic STS/Daily STS based only on actual state;
5. see expected income and obligations in a separate explainable Forecast;
6. understand credit/BNPL/installment spending without duplicate expense recognition;
7. receive deterministic contextual insights when spending pace, funding dependency, or financial state needs attention;
8. see budget projections only after at least 3 Financial Cycle days have elapsed;
9. correct historical data and see balances, budgets, STS, forecast, and insights propagate consistently.

## 8. Non-Goals and Boundaries

V1 excludes cryptocurrency tracking, investment portfolio management, bill splitting/shared household ledgers, tax/general-ledger accounting, open-banking synchronization unless separately scoped, probabilistic income scoring, autonomous money movement, and an LLM financial chatbot.

V1 is not designed for every financial persona. It is a single-user PFM baseline unless another document explicitly expands identity/account ownership scope.
