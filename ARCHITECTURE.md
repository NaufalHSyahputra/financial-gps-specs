# Architecture

This is the v1 implementation direction for the current specification repository; no application or database schema is implied to exist. This document and the other living references are the active baseline; archived specifications preserve historical context. Exact financial behavior lives in [FINANCIAL_RULES.md](FINANCIAL_RULES.md), examples in [TEST_SCENARIOS.md](TEST_SCENARIOS.md), and unresolved choices in [DECISIONS.md](DECISIONS.md). Do not resolve missing financial behavior by inference.

## 1. Topology & Module Boundaries

Android and macOS are first-class v1 targets. Use Kotlin Multiplatform (KMP), Compose Multiplatform, and a **single-process, local-first modular monolith** on each device. SQLite is the local canonical database and SQLDelight provides typed access behind domain-facing ports. iOS, web, multi-device sync, microservices, external brokers and event sourcing are outside this baseline. Backup and restore are user-controlled portability, not sync.

```text
Android / macOS UI and platform adapters
               ↓
        App use-case layer
               ↓
 Calendar | Ledger | Allocation / Budget | Commitments | Income | Intelligence
               ↓                           ↓
    deterministic domain rules       repository ports
                                           ↓
                               SQLDelight → local SQLite
```

Arrows show dependencies, not separate services or a mandated package layout. Shared KMP code owns domain rules, calculation, validation and use-case orchestration. Platform adapters supply clock/instant, selected-zone services, file access, secure facilities and lifecycle integration; they do not own financial rules. Compose UI invokes use cases and displays their results, including unavailable or stale states; it does not calculate authoritative balances.

| Module | Responsibility and boundary |
|---|---|
| **Calendar** | Own configuration facts and derive the unique cycle, cycle type and position for an explicit local financial date. Other modules consume its results rather than reconstruct boundaries. |
| **Ledger / Accounts** | Own accounts, posted/voided realized transaction facts, account effects, opening facts, historical checkpoints, relevant ordering choices and ledger audit. Derive balances from valid facts; distinguish unknown from zero. |
| **Allocation** | Own explicit Flexible, Reserved and Locked intent and obligation links; derive Unallocated and shortfall from actual Available and allocations. Never silently release protected allocations. |
| **Budget** | Own cycle-scoped plans and category mappings. Calculate eligible actual spending, pace and projections; budget changes do not reserve money. Budget and Allocation may share a module, but retain these responsibilities. |
| **Commitments** | Own obligations, recurring and staged rules, statements, installments and settlement validation. Coordinate payments and corrections with Ledger and Calendar. |
| **Income** | Own expected-income occurrences and their lifecycle. Ledger owns posted actual income; an explicit user link coordinates both owners. |
| **Forecast / Advisor** | Compute actual safety guidance and separate projections from authoritative inputs; produce deterministic explanations and insights. Assumptions never mutate actual facts. |
| **App / UI** | Orchestrate commands across owners and render state; keep writes separate from queries logically, without separate databases. Provide user choices, preview, validation and conflict resolution. |

The app obtains an explicit evaluation instant from the platform clock and converts it using the user-selected IANA financial timezone (`Asia/Jakarta`, `Asia/Makassar`, or `Asia/Jayapura`). Calendar's pure resolver receives local dates; it does not read a hidden clock or device timezone. Financial time and audit/recording time are distinct. See [Calendar §§5–10](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md#5-temporal-and-configuration-lifecycle).

## 2. Core Domain Entities & Proposed Data Model

**Proposed, not implemented:** the following are conceptual persistence candidates, not approved SQL tables, columns, aggregate boundaries or migrations. Physical grouping, keys, indexes and cross-module contracts remain open under [Q-04–06 and AUD-LDG-001–009](DECISIONS.md#open--unresolved-questions-and-verification). Use stable identities, appropriate timestamps, provenance and referential integrity. Store canonical facts separately from optional rebuildable read models.

| Candidate table or fact group | Purpose and conceptual relationships |
|---|---|
| `financial_profile` | One local user's configuration: IDR currency, selected financial timezone and relevant settings. |
| `calendar_revision`, `calendar_pending_change` | Configuration history and at most the applicable pending change. Cycles and positions are derived, not independently authoritative rows. |
| `account`, `account_opening` | Account class, status, liquidity eligibility and effective-time opening fact; openings belong to an account and preserve corrected history. |
| `transaction`, `transaction_effect` | Realized event and its account effects. A transfer has one event and source/destination effects; transaction status and effective time are explicit. |
| `category`, `transaction_category` | Spending classification and historical category references; archive does not erase history. Physical mapping cardinality remains a design choice. |
| `allocation`, `reservation_link` | Explicit Flexible / Reserved / Locked amounts and relevant obligation association; Unallocated is derived, not a mandatory allocation record. |
| `budget`, `budget_category` | Cycle-scoped spending plan and category mappings. Cycle identity/representation remains a persistence decision; spending is derived from eligible actual expenses. |
| `obligation`, `recurrence_rule`, `amount_stage` | Required outflow, settlement state and optional generated recurring/staged sources. Stages must not overlap for an occurrence. |
| `credit_statement`, `installment_plan`, `installment` | Liability repayment planning and explicit installment amounts; associated obligations represent due payments, not duplicate purchase expenses. |
| `settlement_link` | Associates an actual payment with an obligation and amount; protect against invalid or duplicate settlement. |
| `expected_income`, `income_link` | Expected occurrence and optional explicit association with a posted actual income transaction; actual income remains valid unlinked. |
| `refund_link` | Explicit association to the original purchase; protect the combined valid linked amount under concurrency. |
| `card_debt_checkpoint`, `financial_order_choice` | Earlier known card debt and user-supplied ordering where materially needed. Their exact persistence, validity and provenance are open. |
| `audit_revision` | Durable before/after or equivalent change evidence and originating operation, including relevant user intent; distinct from events and current canonical state. |
| Optional `derived_cache` / read model | Rebuildable balances, summaries or projections with evaluation time and provenance. Never the sole copy of canonical facts. |

The table names are illustrative. A relationship may require more than one record or a different representation; do not implement an unresolved lifecycle or assume the listed fields settle [Q-04–06](DECISIONS.md#open--unresolved-questions-and-verification). Unknown historical card debt is not zero; earlier card checkpoints are not added to the later opening. Detailed formulas, effective-time rules and examples remain in [FINANCIAL_RULES.md](FINANCIAL_RULES.md).

## 3. Consistency, Atomicity & Audit Model

- A successful financial command commits its canonical changes, required cross-module invariant checks and durable audit together in an **application-level unit of work**. A failed validation, conflict, cancellation or persistence error leaves the pre-command canonical state intact. UI success follows commit.
- Multi-leg transfers and selected transfer fees commit atomically. Linked income claims, payments/settlements, refund associations and corrections coordinate the relevant owners in the same critical consistency boundary. An existing unlinked refund survives a failed later link; a requested combined create-and-link operation commits both or neither.
- Serialize conflicting writes or detect and resolve revisions at commit. Re-read authoritative inputs and revalidate preview, dependencies and ordering choices before committing; stale previews require a new review. Do not rely on UI validation, unprotected read-then-write, handler timing or an assumption of sequential use. Concurrent refund links cannot exceed the original purchase in combination; concurrent allocation changes cannot silently oversubscribe.
- Corrections and voids retain audit and recalculate affected balances, allocation/shortfall, budgets, obligations, STS, forecast and insights as applicable. Prevent orphaned links and silent duplicate settlement, expense or income. Preserve truthful unfavorable expenses while surfacing the resulting state.
- Canonical facts, derived actual state and future projections are distinct. Derived values use an explicit evaluation time, selected timezone and deterministic rules; results must be explainable and rebuildable. The dependency graph must be acyclic. An optimization or cache must agree with a correct rebuild and cannot become financial truth.
- Domain events may report completed facts in process and trigger recoverable noncritical reactions after critical state is consistent. Events are not an event store, an external broker or a substitute for the transaction. Secondary-handler failure cannot undo or misrepresent a committed fact; lagging derived state must be identifiable and recoverable. No transactional outbox is selected for v1.
- The current model must work without replaying audit history. Audit retains the origin and relevant before/after facts and user link/ordering intent; it is not an event-sourced ledger.

**OPEN:** `Q-05 / AUD-LDG-003` includes how an expense draws down Flexible versus Reserved and when unchanged allocations create a shortfall. `AUD-LDG-001` requires refund-concurrency verification; `AUD-LDG-002/004` cover historical debt, temporal boundaries and ordering; `AUD-LDG-005–009` cover other validation, unlinked income, unsupported card credit, reads/rebuild and reconciliation. Consult [DECISIONS.md](DECISIONS.md#open--unresolved-questions-and-verification) before implementing an affected path; the owner-frozen Ledger baseline is not evidence that these findings or tests are closed.

## 4. Critical Lifecycle & State Transitions

| State | Transition and invariant |
|---|---|
| **Cycle** | A configuration change protects the entire active cycle. From the next boundary, resolve the new adjusted boundary first: a transition **<15 days** merges with the next regular cycle as `EXTENDED`; **≥15 days** is a standalone `SHORT` cycle. Pending settings can be replaced or cancelled before they become effective; effectiveness follows time, not a scheduler. Calendar history before the initial floor is unavailable. |
| **Expected income** | `EXPECTED → RECEIVED` only by an explicit link to posted actual income; `EXPECTED → MISSED` when expected time passes without that link; `MISSED → RECEIVED` requires an explicit late link. Cancellation is supported; exceptional unlink/correction outcomes remain subject to `Q-05`. Date-only expectations pass after the end of their local financial date. |
| **Transaction** | A valid actual event is `POSTED`; an approved void changes it to `VOIDED`, removes its current financial effects and preserves audit and dependent consistency. Corrections are auditable and propagate; do not destructively erase dependent financial history. Physical correction representation is open under `Q-04–06`. |
| **Allocation shortfall** | Normal explicit allocation changes satisfy allocated ≤ Available. A truthful actual change may make allocated > Available: retain the fact, surface `ALLOCATION_SHORTFALL`, and make authoritative STS and Daily STS `UNRESOLVED` until explicit resolution. Expense drawdown is open under `Q-05 / AUD-LDG-003`; do not invent its allocation effects. |
| **Opening and ordering** | Asset openings are effective-time snapshots: earlier activity remains historical but is not applied twice. Where same-day or equal-time order changes a result, record the user's applicable ordering; unknown required order is indeterminate. Historical card checkpoints are compared with, not added to, later openings. Exact boundary and revalidation contracts remain open under `AUD-LDG-002/004`. |

## 5. Non-Functional, Privacy & Recovery Baseline

- **Money and time:** IDR only in v1. Store and calculate canonical money as integer currency minor units, never floating point; apply explicit engine rounding. Entered transaction amounts are positive, with direction represented by transaction semantics; other domain amounts follow their specific validation. Use local financial dates and the configured IANA timezone for cycle behavior.
- **Offline and integrity:** Local canonical writes and critical validation do not depend on a network, background handler or later synchronization. A failed mutation leaves state unchanged. Version and test migrations against representative prior data; protect referential integrity and reject incompatible data safely.
- **Recovery:** Support explicit export, import, backup and restore. Validate version, compatibility and integrity before any authorized replacement or merge; failed restore must not corrupt existing canonical data. Preserve facts, effective times, relationships, ordering choices and audit as required. Test restore, not just backup creation. Format, merge/replacement and conflict behavior remain open under `Q-08`. Do not sync or copy an open live SQLite database as a sync strategy.
- **Security:** Encrypt sensitive user and financial data at rest using appropriate platform mechanisms. Validate imported and external data at the trusted boundary, use least-privilege file access, keep secrets out of source control, assess dependencies and avoid sensitive data in logs or crash reports. Any future client-server traffic containing sensitive data requires TLS 1.2+; authentication and authorization requirements apply if such a deployment is introduced.
- **Privacy and analytics:** Collect only necessary data. By default analytics must not send exact balances, STS, transaction/income/obligation amounts, account or income-source names, merchant descriptions, free text, card/account numbers, credentials or precise location. Prefer categorical or bucketed properties and pseudonymous identifiers that are not authorization identifiers. Analytics is separate from operational diagnostics; disabling it or a failure must never block financial operations. Any `expected_income_linked_received` event follows an actual explicit link, never similarity or date arrival.
- **Correctness and accessibility:** Run golden financial fixtures and relevant atomicity, concurrency, migration and restore tests; compare shared behavior across supported platforms. Never present a newly failed calculation as authoritative; a last-known value must be clearly stale and time-stamped. Do not convey financial state by color alone; controls need accessible names, sensible focus and keyboard operation where applicable.
- **Performance:** Preserve correctness first. Existing targets include ordinary validation within 100 ms, typical engine recalculation within 200 ms at p95 and a meaningfully usable dashboard within 2.5 s at p75; measurement conditions remain open under `Q-09`. Design for at least 10,000 transactions, 1,000 obligations and 1,000 expected-income/forecast events without losing correctness, with graceful degradation.

Sources: [architecture constitution](docs/archive/architecture/ARCHITECTURE_CONSTITUTION.md), [strategic DDD](docs/archive/architecture/strategic-ddd.md), [Calendar](docs/archive/architecture/financial-calendar-tactical-ddd-v1.0.md), [Ledger](docs/archive/architecture/financial-accounts-ledger-tactical-ddd-draft.md), [domain model](docs/archive/design/08-domain-model.md), [non-functional requirements](docs/archive/requirements/07-non-functional-requirements.md), and [analytics](docs/archive/reference/12-analytics.md).
