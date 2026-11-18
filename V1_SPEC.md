# Financial GPS — V1 Product Specification

## Core Problem & Product Promise

People need more than a record of where their money went: they need to know what is available now, what is safe to spend before their next cycle, and whether future cash can cover upcoming commitments. A balance alone hides protected money, obligations and liabilities.

Financial GPS is a single-user personal finance application for a salaried person who may also have variable income, multiple accounts and credit obligations. It connects past transactions, present financial state and future plans. It keeps allocation explicit, separates actual money from forecast assumptions, and provides deterministic, explainable Safe to Spend (STS) guidance. The founder is the first user, not the only supported persona; payday, categories and providers must not be hardcoded to one person's finances.

The core loop is **record actual activity → understand balances, allocations and budgets → see current STS → inspect forecast and contextual guidance → choose an action**. Future income cannot increase current STS. Newly available money stays Unallocated until the user explicitly assigns its role. Reserved and Locked money are not silently released. Financial rules and open implementation questions are in [FINANCIAL_RULES.md](FINANCIAL_RULES.md) and [DECISIONS.md](DECISIONS.md).

## In-Scope V1 Capabilities

### Setup & Financial Calendar

- [ ] Single-user profile and short onboarding: IDR, selected Indonesian financial timezone, cycle/payday anchor, weekday adjustment policy and starting Cash; additional accounts are optional later.
- [ ] Resolve the active Financial Cycle using `NONE`, `PREVIOUS_WORKING_DAY` or `NEXT_WORKING_DAY`; no holiday calendar. A configuration change preserves the active cycle and uses the approved Short/Extended transition rules.
- [ ] Keep starting Cash Unallocated by default and show Available and Unallocated even when STS is zero.

### Accounts & Ledger

- [ ] Maintain asset and liability accounts, including cash, bank, e-wallet, savings, credit card, PayLater and supported generic types; archive without erasing history or eligible balance.
- [ ] Manually record income, expense, transfer, liability purchase/payment, refund and explained balance adjustment. Provide calculator-first quick expense entry; optional details do not block ordinary capture.
- [ ] Show actual account activity and liabilities; transfers are not spending, transfer fees are separately recognized, and credit/BNPL repayments do not repeat purchase expense.
- [ ] Support auditable edits, voids and guided corrections, with dependent effects recalculated atomically. Preserve historical entry, opening-snapshot and ordering behavior; surface unknown or unsupported card credit instead of inventing a result.

### Allocation & Budgets

- [ ] Explicitly allocate actual available money to Flexible, Reserved or Locked, or leave it Unallocated. Surface allocation shortfall and mark STS unresolved until the user resolves it; never silently choose what to release.
- [ ] Create cycle-scoped category or purpose budgets and display amount, spent, remaining/overspend, limit status, cycle progress and pace. A budget is a spending plan, not reserved money.
- [ ] Allow total active budgets to exceed Flexible with a non-blocking warning showing the difference; budget changes move no money.
- [ ] Show deterministic spending and overspend projections only from cycle day 3 when required history is sufficiently complete. On a new cycle, start with no budgets; a user may apply prior definitions as new instances with zero spent.

### Obligations & Income

- [ ] Track manual, recurring and staged obligations, credit statements, BNPL and installment payments, including due dates and settlement. Staged amounts resolve by effective period.
- [ ] Plan stable and uncertain Expected Income separately from actual income. Posting actual income is valid without a link; only an explicit user-selected link marks an eligible expected occurrence `RECEIVED`. Newly available actual money stays Unallocated until explicitly assigned.
- [ ] Show missed or late expected income and recalculate future funding risk without silently creating an actual receipt.

### Safety Guidance, Forecast & Insights

- [ ] Explain actual-only STS and Daily STS using Flexible money and applicable unfunded obligations through the active cycle. Exclude Unallocated, protected money, expected income and post-cycle obligations from current STS.
- [ ] Show a deterministic forecast from actual starting state, with stable expected income in the Base Forecast and uncertain income only in user-selected scenarios. Label future assumptions and never mutate actual state.
- [ ] Show future funding dependency when an obligation lacks sufficient actual-only funding and depends on stable expected income before it; update the risk when income is missed, delayed or actual funding changes.
- [ ] Provide contextual deterministic insights in Home, Budget and Planning. Numeric guidance comes from engine outputs; omit unavailable numbers and require explicit user action for any change.
- [ ] Support user-controlled export, import, backup and restore, with validation and safe handling of incompatible data.

## Not Doing Now

- Crypto tracking, investment portfolio valuation, shared ledgers or bill splitting, and advanced accounting or tax.
- Automatic bank/open-finance synchronization, live multi-device sync, or multi-currency consolidation and FX.
- Public-holiday calendar dependencies, user-configurable budget warning thresholds, or a complex automatic income-allocation builder.
- Probabilistic/ML forecasts, conversational LLM financial advice, or autonomous transfers, borrowing and savings actions.
- Provider-specific rules embedded in the financial engine, or full financial-life setup as an onboarding prerequisite.

## Navigation, Core User Flows & UI State Map

Primary navigation: **Home | Transactions | Budget | Accounts | Planning**. Insights are contextual rather than a required primary tab.

| Area | Core flow and state |
|---|---|
| **Onboarding** | Choose timezone, anchor and adjustment → enter starting Cash → enter the app with money Unallocated. |
| **Home** | See STS or its unresolved/deficit state, Daily STS, cycle, budget attention and obligations. “Why this amount?” explains Flexible, applicable obligations and Reserved coverage, and identifies excluded expected income and Unallocated money. |
| **Transactions** | `+ → amount → category → account → save`; after a valid save, recalculate affected actual and derived state. For a transfer choose different accounts; an optional fee is separate. |
| **Accounts & allocation** | Review assets and liabilities; explicitly assign or reallocate money. If allocations exceed Available, show the shortfall and a resolution path instead of an authoritative STS. |
| **Budget** | Create a cycle plan and review actual spending, status and pace. Warn when plan exceeds Flexible; offer an explicit prior-cycle template when the new cycle has no budgets. |
| **Planning** | Add expected income and obligations, compare Base and selected scenarios, and inspect future funding dependency and projected risk. Label actual starting state separately from assumptions. |
| **Corrections** | Preview a change, surface affected links or ordering, obtain required choices, then commit valid effects together or preserve the existing facts. |

The following existing HTML prototypes and their paired screenshots are visual references; their illustrative content does not override product or financial rules.

### Home states

| State | HTML prototype | Screenshot |
|---|---|---|
| On track | [Open](docs/design/assets/home_on_track/code.html) | [View](docs/design/assets/home_on_track/screen.png) |
| Budget off track | [Open](docs/design/assets/home_budget_off_track/code.html) | [View](docs/design/assets/home_budget_off_track/screen.png) |
| Low Safe to Spend | [Open](docs/design/assets/home_low_safe_to_spend/code.html) | [View](docs/design/assets/home_low_safe_to_spend/screen.png) |
| Funding dependency | [Open](docs/design/assets/home_funding_dependency/code.html) | [View](docs/design/assets/home_funding_dependency/screen.png) |
| Missed expected income | [Open](docs/design/assets/home_missed_expected_income/code.html) | [View](docs/design/assets/home_missed_expected_income/screen.png) |
| Unallocated money | [Open](docs/design/assets/home_unallocated_money/code.html) | [View](docs/design/assets/home_unallocated_money/screen.png) |

### Screens and flows

| Screen | HTML prototype | Screenshot |
|---|---|---|
| Accounts | [Open](docs/design/assets/accounts/code.html) | [View](docs/design/assets/accounts/screen.png) |
| Add transaction | [Open](docs/design/assets/add_transaction/code.html) | [View](docs/design/assets/add_transaction/screen.png) |
| Allocate money | [Open](docs/design/assets/allocate_money/code.html) | [View](docs/design/assets/allocate_money/screen.png) |
| Budget | [Open](docs/design/assets/budget/code.html) | [View](docs/design/assets/budget/screen.png) |
| Planning | [Open](docs/design/assets/planning/code.html) | [View](docs/design/assets/planning/screen.png) |
| Forward planning workspace | [Open](docs/design/assets/planning_forward_financial_workspace/code.html) | [View](docs/design/assets/planning_forward_financial_workspace/screen.png) |
| Transactions | [Open](docs/design/assets/transactions/code.html) | [View](docs/design/assets/transactions/screen.png) |

### Essential UI conventions

- Distinguish Balance, Available, Flexible, Budget Remaining and STS; label actual, expected and scenario values in text as well as visually. Do not present credit capacity as owned cash.
- Provide clear empty, loading, validation, error, success and unavailable states. Preserve input on recoverable errors; explain what the user can do next. Do not show a fabricated projection or treat stale guidance as current.
- Use accessible control names, visible focus, keyboard operation where applicable, readable contrast and understandable currency/date formatting. Never rely on color or icons alone for financial state; respect reduced-motion and platform accessibility preferences.
- Use the archived [design system](docs/archive/design/DESIGN.md) for historical visual guidance and the living prototypes for visual detail.

Source material: [vision](docs/archive/product/01-product-vision.md), [PRD](docs/archive/product/02-v1-prd.md), [scope](docs/archive/product/03-v1-scope.md), [information architecture](docs/archive/design/09-information-architecture.md), and [user flows](docs/archive/design/10-user-flows.md).
