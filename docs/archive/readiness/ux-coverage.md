# UX coverage review

**Status:** Preliminary inventory, not design or usability approval. Screen assets are illustrative and subordinate to [flows](../design/10-user-flows.md), [information architecture](../design/09-information-architecture.md), and [design system](../design/DESIGN.md). Asset existence is not evidence that all interactions, accessibility requirements or mobile/desktop states work.

| Critical flows | Existing asset folders | Coverage and review work |
|---|---|---|
| Onboarding and cycle settings: UF-ONB-001, UF-CYCLE-001 | No dedicated asset found | Design initial, invalid input, timezone and cycle-change preview/transition states. |
| Transactions: UF-TRX-001, UF-TRX-002, UF-EDIT-001 | `transactions`, `add_transaction` | Review quick input, transfers, invalid input, edit/void, dependencies, success and accessibility. No separate correction flow asset found. |
| Accounts, liabilities and credit: UF-CC-001, UF-BNPL-001 | `accounts`, `transactions` | Review account creation/archive, liability details, statement/payment and error states; no dedicated end-to-end credit/BNPL asset found. |
| Allocation: UF-INC-002, UF-ALLOC-001 | `allocate_money`, `home_unallocated_money` | Review explicit destinations, partial allocation, shortfall resolution, empty/error and focus states; no dedicated shortfall asset found. |
| Budget: UF-BUD-001–003 | `budget`, `home_budget_off_track` | Review create, warning, day 1–2, incomplete data, cycle close/template and correction states; no dedicated flow assets found. |
| STS and guidance: UF-STS-001, UF-ADV-001 | `home_on_track`, `home_low_safe_to_spend`, `home_unallocated_money`, `home_budget_off_track` | Review explanation, deficit, unresolved allocation and assistive-technology presentation. Asset names alone do not prove coverage. |
| Planning: UF-INC-001, UF-FCST-001–002, UF-OBL-001 | `planning`, `planning_forward_financial_workspace`, `home_funding_dependency`, `home_missed_expected_income` | Review create/edit/receive income, scenarios, obligations, missed income, dependency and error/empty states; no dedicated editor assets found. |

For every row: inspect the actual mockup against all referenced flows and NFR-A11Y requirements, record desktop/mobile differences, keyboard and screen-reader behavior, and link review evidence. Confirm loading, empty, invalid, failure, correction and success states where applicable. Run the relevant research tasks in [assumptions](assumptions-research.md); resolve Q-07 before marking a UI slice ready.
