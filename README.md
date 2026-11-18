# Financial GPS

Financial GPS is a language- and stack-agnostic specification for a local-first personal finance application that records actual transactions, explicitly allocates available money, explains deterministic Safe to Spend, and forecasts future cash flow without treating expected income as cash already held.

## Start here

This repository currently contains a **specification-only baseline**; application implementation has not yet started. Before coding an affected slice, read the relevant financial rules and check [DECISIONS.md](DECISIONS.md) for `OPEN` questions and verification boundaries. In particular, expense allocation drawdown and Ledger integration contracts remain unresolved.

| Living reference | Purpose |
|---|---|
| [V1_SPEC.md](V1_SPEC.md) | Product problem, v1 scope, user flows and UI states. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Topology, proposed data model, consistency and recovery. |
| [FINANCIAL_RULES.md](FINANCIAL_RULES.md) | Financial definitions, formulas, invariants and examples. |
| [TEST_SCENARIOS.md](TEST_SCENARIOS.md) | Given / When / Then scenarios and known blockers. |
| [DECISIONS.md](DECISIONS.md) | Approved decisions and unresolved questions. |
| [WORKFLOW.md](WORKFLOW.md) | Solo feature loop and documentation guardrails. |
| [Design assets](docs/design/assets/) | Living HTML prototypes and paired screenshots. |

## Historical specifications

Earlier specifications and the pre-migration README are preserved in [docs/archive/](docs/archive/). The living references above are the active baseline.
