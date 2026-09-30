# Architecture Constitution v1.0

## 1. Document Metadata

| Field | Value |
|---|---|
| Project | Expense Budget Tracker / Personal Financial Management (PFM) |
| Document | Architecture Constitution |
| Version | 1.0 |
| Status | APPROVED / FROZEN |
| Effective scope | V1 application architecture and all implementation work derived from it |
| Last updated | 2026-09-29 |
| Normative language | **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are requirements with the meanings defined in Section 2.2 |

### 1.1 Decision labels

- **LOCKED V1** identifies a decision that governs V1 implementation and MUST NOT be changed implicitly.
- **DEFERRED/FUTURE** identifies an idea that is not part of V1 and MUST NOT be treated as an approved V1 requirement.
- A deferred idea MAY be investigated, but it MUST pass the change governance in Section 21 before it can alter this constitution or the implementation baseline.

## 2. Purpose and Authority

### 2.1 Purpose

This constitution is the canonical normative architecture document for the Expense Budget Tracker / PFM project. It constrains implementation choices, architectural reviews, and AI coding-agent work while preserving space for later design phases.

The architecture MUST serve the product first and learning goals second. A technology or pattern MUST NOT be introduced only for novelty, résumé value, or architectural experimentation. Learning MAY influence a choice only when that choice remains appropriate for the product, scope, risk, and maintenance cost.

### 2.2 Normative terms

- **MUST / MUST NOT**: mandatory for compliance.
- **SHOULD / SHOULD NOT**: expected unless a documented, reviewed reason justifies an exception.
- **MAY**: permitted but not required.

### 2.3 Authority

Implementations, technical designs, generated code, and agent instructions MUST comply with this constitution. An implementation convenience, framework default, generated artifact, or AI suggestion MUST NOT silently override it.

Where this document intentionally leaves a design choice open, a later design artifact MAY decide it without amending this constitution, provided that the decision does not conflict with any locked rule.

## 3. Relationship to Other Canonical Sources

This constitution does not replace the locked **Product Requirements**, **Financial Engine**, **UX Foundation**, or **Design System**. Those sources remain authoritative within their respective domains.

The following precedence rules apply:

1. Locked product behavior and financial semantics MUST come from the locked Product Requirements and Financial Engine sources.
2. Locked user experience and visual behavior MUST come from the locked UX Foundation and Design System sources.
3. Architecture and implementation structure MUST comply with this constitution.
4. Later technical designs, ADRs, source code, tests, and agent output MUST comply with all applicable locked sources above.

This precedence is complementary, not permission to reinterpret one source through another. If two locked sources appear to conflict, implementation MUST pause at the conflicting point and raise a Change Request; it MUST NOT silently choose a winner or redefine the requirement.

This document MUST NOT invent, broaden, narrow, or otherwise change locked product, financial, UX, or design-system requirements.

## 4. Architectural Goals — LOCKED V1

The V1 architecture MUST:

- support Android and macOS as first-class platforms from a shared Kotlin codebase where appropriate;
- operate local-first without requiring an account, authentication, network connection, or backend service;
- preserve financial correctness, determinism, auditability, and explainability;
- keep canonical financial facts clearly separated from derived current state and future projections;
- make derived state rebuildable from canonical facts and applicable deterministic rules;
- support maintainable evolution through explicit module boundaries and repository abstractions;
- remain testable at domain, persistence, state-transition, and architecture-boundary levels; and
- remain proportionate to a V1 personal finance product.

## 5. Explicit Non-Goals — LOCKED V1

V1 MUST NOT include or assume:

- iOS or Web delivery;
- accounts, user authentication, or identity infrastructure;
- a required backend or always-online operation;
- multi-device synchronization;
- distributed messaging infrastructure;
- event sourcing;
- a transactional outbox;
- microservices;
- live synchronization of a SQLite database file; or
- direct AI authority over financial calculations or persisted state.

Export, import, backup, and restore are V1 data portability and recovery capabilities. They MUST NOT be represented as multi-device synchronization.

## 6. Platform and Technology Baseline — LOCKED V1

- Android and macOS desktop MUST be treated as first-class V1 platforms.
- iOS and Web are **DEFERRED/FUTURE**.
- The application MUST use Kotlin Multiplatform (KMP) and Compose Multiplatform.
- Platform parity SHOULD be pursued for locked product behavior, financial semantics, and core workflows unless a locked platform-specific requirement states otherwise.
- Platform limitations MUST be surfaced explicitly; they MUST NOT be hidden by changing domain behavior.

Adopting iOS, Web, a different application framework, or a different primary language requires governance under Section 21.

## 7. Application Topology — LOCKED V1

The application MUST be a **modular monolith**.

- Modules MUST have explicit responsibilities and controlled dependencies.
- Cross-module interaction SHOULD occur through deliberate contracts rather than incidental access to internals.
- Dependency direction MUST protect the domain and deterministic financial calculation core from UI, storage, and platform concerns.
- The monolith MAY use in-process events for decoupled secondary reactions.
- The implementation MUST NOT introduce microservices, network service boundaries, or distributed infrastructure to simulate future scale.

This constitution does not define module names, package/folder structure, bounded contexts, or deployment subcomponents. Those belong to later design work.

## 8. Domain Model and Command/Query Separation — LOCKED V1

The architecture MUST use pragmatic Domain-Driven Design (DDD) with a rich domain model.

- Financial invariants and business rules MUST be represented and enforced in the domain rather than scattered across UI, persistence adapters, or presentation code.
- Domain objects MUST protect valid state transitions; they MUST NOT be reduced to passive data containers when behavior or invariants belong with the model.
- DDD techniques MUST be applied pragmatically and MUST NOT force unnecessary ceremony.

The application MUST use **logical CQRS**:

- commands and state-changing use cases MUST be conceptually separated from queries and read concerns;
- this separation MAY exist within one process and one database;
- logical CQRS MUST NOT be interpreted as requiring separate services, separate physical databases, asynchronous writes, or event sourcing.

Concrete bounded contexts, aggregates, command catalogs, and query catalogs are deliberately deferred to later design phases.

## 9. Domain Events — LOCKED V1

The architecture MUST support meaningful domain events for completed domain occurrences.

- Domain event names MUST use meaningful past-tense language.
- An event MUST represent something that has happened, not an instruction disguised as an event.
- Events SHOULD carry the minimum stable information required by their consumers.
- Events MAY trigger in-process secondary reactions after the financially critical state is made consistent.
- Reactions SHOULD be idempotent where duplicate handling is possible or where retry/re-entry can occur.
- Event handling MUST NOT become the sole hidden location of financially critical invariants.

The application is **event-driven where useful, but explicitly not event-sourced**. Current state MUST NOT depend on replaying an event log as the system of record. This constitution does not define a detailed event catalog; that is later design work.

## 10. Financial State Model — LOCKED V1

The design MUST explicitly distinguish:

1. **Canonical facts**: authoritative recorded inputs and financial occurrences.
2. **Derived actual state**: current results deterministically computed from canonical facts and applicable rules.
3. **Future projections**: forward-looking results based on plans, expectations, schedules, or assumptions rather than completed facts.

These categories MUST NOT be conflated in persistence, calculation, presentation, or audit behavior. A projection MUST NOT be presented or persisted as a completed financial fact. Derived state MUST NOT silently become an independent source of truth.

## 11. Financial Calculation Core — LOCKED V1

- Financial calculations MUST be deterministic: the same canonical inputs, rules, and relevant time-zone context MUST produce the same result.
- Calculation logic SHOULD be pure where practical.
- The calculation dependency graph MUST be explicit and acyclic.
- Cyclic calculation dependencies MUST be rejected by design or detected by automated verification.
- Derived state MUST be rebuildable from canonical facts and deterministic rules.
- Correctness MUST be established before incremental optimization.
- Performance optimizations MUST preserve observable financial semantics and MUST be verifiable against a correct rebuild.
- Hidden dependence on wall-clock time, platform locale, iteration order, or mutable global state MUST NOT influence financial results.

## 12. Consistency and Reactions — LOCKED V1

Financially critical state MUST be immediately consistent at the completion boundary of its state-changing use case. A successful operation MUST NOT expose a partially applied financial result.

The application MUST use a hybrid consistency model:

- canonical writes and financially critical invariant enforcement MUST complete synchronously and atomically where required;
- non-critical secondary reactions MAY run asynchronously or later within the same process;
- failure of a secondary reaction MUST NOT retroactively misrepresent a committed canonical fact;
- secondary state that can lag MUST be identifiable and recoverable; and
- correctness MUST NOT depend on timing luck or handler execution order.

## 13. Persistence — LOCKED V1

- SQLite MUST be the V1 persistence database.
- SQLDelight MUST be used for typed database access.
- Persistence MUST sit behind repository ports or equivalent domain-facing abstractions.
- Domain and financial calculation code MUST NOT depend directly on SQLite or SQLDelight implementation details.
- Transaction boundaries MUST preserve financially critical consistency.

This constitution does not define an ERD, concrete tables, columns, indexes, migrations, or SQL queries. Those decisions belong to later persistence design.

### 13.1 Derived caches and read models

- A derived cache or read model MUST be reproducible from canonical facts and applicable deterministic rules.
- It MUST NOT become the only copy of a canonical fact.
- Its provenance and rebuild mechanism SHOULD be explicit.
- Stale or missing derived data MUST be recoverable without corrupting canonical facts.
- Cache invalidation or read-model update failures MUST NOT silently alter financial truth.
- A derived representation MAY be persisted for performance only after correctness has been demonstrated without relying on it.

## 14. In-Process Event Infrastructure — LOCKED V1

- Domain event dispatch MUST be in-process.
- The dispatcher SHOULD make handler registration, failure behavior, and execution semantics explicit.
- Kafka, RabbitMQ, and comparable external message brokers MUST NOT be introduced in V1.
- A transactional outbox MUST NOT be introduced in V1.
- In-process asynchronous work MUST remain compatible with local-first operation and application restarts; canonical correctness MUST NOT rely on volatile queued work.

External brokers or an outbox MAY be reconsidered only if an actual future requirement creates a justified distributed-delivery problem.

## 15. Explainability and Deterministic Rules — LOCKED V1

Explainability MUST be built into the financial engine rather than reconstructed from opaque outputs.

- Significant financial results SHOULD be traceable to canonical inputs, applied deterministic rules, and intermediate derivations sufficient for human review.
- Rule evaluation order and conflict resolution MUST be deterministic.
- Equivalent inputs MUST NOT produce different results because of platform, execution timing, or non-deterministic collection ordering.
- Explanations MUST reflect the actual calculation path; they MUST NOT be generated as an unrelated narrative that can disagree with the result.
- Optimization MUST preserve the ability to explain and test outcomes.

## 16. Money, Currency, and Time — LOCKED V1

- V1 MUST support Indonesian rupiah (IDR) only.
- Financial amounts MUST use a `Money` value object or equivalent domain value type; raw floating-point values MUST NOT represent money.
- Currency assumptions and rounding behavior MUST be explicit and deterministic in the later Financial Engine design and implementation.
- Users MUST be able to select one of these Indonesian IANA time zones:
  - `Asia/Jakarta`
  - `Asia/Makassar`
  - `Asia/Jayapura`
- Financial date/time interpretation MUST use the selected IANA time-zone identifier where user-local time is relevant.
- Code MUST NOT replace these zones with fixed UTC offsets or infer them from ambiguous abbreviations.

Additional currencies and additional time zones are **DEFERRED/FUTURE** and require approved requirements and architecture review.

## 17. Audit History Without Event Sourcing — LOCKED V1

The application MUST preserve audit history appropriate to locked product and financial requirements.

- Audit records MUST make relevant changes attributable to their originating operation and resulting state transition to the extent required by the locked requirements.
- Audit data MUST be distinct from domain-event delivery mechanics.
- Audit history MUST NOT be treated as an event-sourced system of record.
- The current authoritative model MUST remain usable without replaying audit records.
- Audit retention and presentation details MUST be defined in later design without weakening the required history.

## 18. Security Baseline — LOCKED V1

The application MUST follow a standard security baseline proportionate to a local personal-finance application.

- All external and imported data MUST be validated before it can affect domain state.
- File access and platform capabilities MUST use least privilege.
- Sensitive financial data MUST NOT be written to logs, diagnostics, crash reports, or analytics by default.
- Secrets, if any are introduced by an approved future capability, MUST NOT be hard-coded or committed to source control.
- Dependencies MUST be maintained deliberately, and known relevant vulnerabilities MUST be assessed before release.
- Backup, export, import, and restore flows MUST protect integrity and MUST fail safely on invalid or incompatible input.
- Error messages MUST avoid unnecessary disclosure of sensitive financial data.

This baseline does not add accounts, authentication, cloud storage, telemetry, or a backend to V1.

## 19. Backup, Export, Import, and Restore — LOCKED V1

- V1 MUST support export, import, backup, and restore as defined by the locked product requirements.
- A restore MUST validate compatibility and integrity before replacing or merging authoritative local state according to the later approved design.
- These capabilities MUST be described as explicit user-controlled data portability and recovery operations.
- They MUST NOT be marketed, designed, or implemented as live multi-device synchronization.
- The implementation MUST NOT naively synchronize a live SQLite file through Syncthing or any other file-level synchronization mechanism.
- Documentation MUST NOT recommend copying or syncing an open database file as a synchronization strategy.

The exact archive format, versioning scheme, merge behavior, and conflict policy belong to later design and MUST remain consistent with canonical-fact and audit requirements.

## 20. Shared and Platform-Specific Responsibilities — LOCKED V1

KMP shared code SHOULD own behavior that must remain semantically identical across Android and macOS, including:

- domain rules and invariants;
- deterministic financial calculations;
- command/use-case orchestration that is not platform-dependent;
- repository ports and persistence-facing contracts;
- validation and rule evaluation;
- canonical shared models and value objects; and
- cross-platform tests and golden financial scenarios.

Platform-specific code SHOULD own capabilities that genuinely depend on the operating system or platform framework, including platform lifecycle integration, native file pickers, permissions, secure platform facilities, windowing, and other OS integrations.

- Platform adapters MUST conform to shared contracts.
- Business rules MUST NOT be duplicated in platform code merely for convenience.
- Shared code MUST NOT absorb platform-specific abstractions that distort the domain.
- UI code MAY differ where platform conventions require it, but MUST preserve locked product, financial, UX, and design-system behavior.

This guidance does not prescribe package names, source-set layout, or folder structure.

## 21. AI Extension Boundary — LOCKED V1 BOUNDARY; FEATURE DEFERRED/FUTURE

AI/LLM capability is an optional future extension point, not a required V1 feature.

If introduced through approved governance:

- an LLM MAY transform user language into structured intent or a transaction draft;
- LLM output MUST be treated as untrusted proposed input;
- an LLM MUST NOT perform authoritative financial calculations;
- an LLM MUST NOT write directly to canonical persistence or otherwise hold state-change authority;
- every proposed state change MUST pass deterministic domain validation; and
- every proposed state change MUST receive explicit user confirmation before commitment.

The deterministic domain remains the sole authority for validity and financial outcomes. AI-generated explanations MUST NOT replace explanations produced from the actual deterministic calculation path.

## 22. Multi-Device Synchronization — DEFERRED/FUTURE

Multi-device synchronization is not a V1 requirement and MUST NOT be pre-built speculatively.

If multi-device synchronization becomes an actual approved product requirement, a Spring Boot synchronization service MAY be evaluated. Its mention here is an allowed future direction, not a selected design or authorization to implement it.

Before adoption, the future design MUST define ownership, identity, conflict resolution, offline behavior, security, protocol/API contracts, migration, and operational responsibilities through approved requirements, ADRs, and Change Requests. It MUST NOT use naive live SQLite file synchronization.

## 23. Verification Strategy — LOCKED V1

### 23.1 Required test layers

- Domain rules and financial calculations MUST have unit tests.
- Persistence adapters, transaction behavior, and repository contracts MUST have integration tests.
- Financially meaningful state transitions MUST have state-machine tests.
- The project MUST maintain human-readable golden financial scenarios that state inputs, rule context, expected results, and material explanations.
- Architecture fitness tests MUST enforce critical boundaries and prohibited dependencies.
- Property-based tests SHOULD be used where they provide valuable coverage of invariants, arithmetic properties, ordering, idempotency, or broad input spaces.

### 23.2 Rebuild and determinism proof

- Tests MUST demonstrate that applicable derived state can be rebuilt from canonical facts and deterministic rules.
- Golden scenarios MUST run consistently across supported platforms where the shared core is expected to behave identically.
- Optimized or incremental calculations MUST be checked against a correct full calculation or rebuild for representative and boundary scenarios.
- Defect fixes in financial behavior MUST add a regression scenario at the narrowest useful level.

## 24. Prohibited Architecture Drift and Anti-Patterns

The following are non-compliant unless this constitution is formally changed:

- introducing a required backend, account system, or authentication into V1;
- treating iOS, Web, or multi-device sync as already approved V1 scope;
- decomposing the modular monolith into microservices;
- introducing Kafka, RabbitMQ, another external broker, or a transactional outbox in V1;
- implementing event sourcing or making event replay necessary to reconstruct authoritative current state;
- conflating canonical facts, derived actual state, and future projections;
- persisting derived data as an unrebuildable independent source of truth;
- hiding financial rules in UI code, SQL, event handlers, or platform adapters;
- allowing cyclic or implicit financial calculation dependencies;
- accepting eventually consistent financially critical state;
- using floating-point primitives as monetary values;
- using file-level synchronization, including Syncthing, on a live SQLite database;
- allowing an LLM direct calculation authority, persistence access, or unconfirmed writes;
- duplicating shared business logic separately per platform;
- coupling the domain directly to SQLDelight, SQLite, Compose, or operating-system APIs;
- optimizing incrementally before correctness and rebuild equivalence are established;
- creating speculative abstractions or infrastructure solely for learning or hypothetical scale; or
- allowing code generation or agent output to redefine locked requirements.

## 25. Change Governance

### 25.1 ADRs

An Architecture Decision Record (ADR) SHOULD document a significant implementation decision that fits within this constitution. An ADR MAY choose among permitted options, explain trade-offs, and record consequences. An ADR MUST NOT override a locked constitutional rule.

### 25.2 Change Requests

A Change Request is required when a proposal would:

- alter a **LOCKED V1** decision;
- promote a **DEFERRED/FUTURE** idea into approved scope;
- conflict with a locked Product Requirement, Financial Engine, UX Foundation, or Design System rule;
- add a prohibited technology or topology; or
- change the authority, consistency, persistence, security, or AI boundaries defined here.

A Change Request MUST state the motivation, impacted locked sources, alternatives, product value, risks, migration and rollback implications, test impact, and resulting documentation updates. Approval MUST be explicit. Silence, merged code, framework defaults, or an AI-generated recommendation do not constitute approval.

Once approved, the affected canonical documents MUST be versioned and updated before or together with implementation. A superseded rule MUST remain traceable through document history or its associated ADR/Change Request.

## 26. Implementation Consequences

Implementation teams and coding agents MUST:

- begin from locked product and financial behavior, then apply this architecture;
- keep the deterministic domain and calculation core independent of UI, platform, and persistence frameworks;
- define explicit contracts at module and adapter boundaries;
- establish transaction boundaries around financially critical changes;
- treat events as completed facts used for meaningful in-process reactions, not as commands or an event store;
- design derived data with a verified rebuild path;
- include explainability outputs in the calculation design where required, not as a later cosmetic layer;
- keep Android and macOS semantics aligned through shared tests;
- record significant permitted design choices in ADRs; and
- stop and raise a Change Request rather than silently implementing a constitutional exception.

Implementation teams and coding agents MAY select concrete names, algorithms, module boundaries, and internal APIs during later design, but only within these constraints and the other locked canonical sources.

## 27. Deliberately Deferred Design Detail

This constitution intentionally does **not** define:

- entity-relationship diagrams (ERDs);
- concrete database tables, columns, keys, indexes, or migrations;
- API endpoints or network protocols;
- package or folder structure;
- bounded contexts;
- aggregates or aggregate boundaries;
- a detailed command/query catalog;
- a detailed domain-event catalog;
- exact module names and dependency maps;
- backup archive format or merge algorithm; or
- a synchronization service design.

These details MUST be developed in later design phases. They MUST NOT be inferred as locked merely because an example, prototype, generated file, or agent response contains them.

## 28. Architecture Compliance Checklist

Before a design or implementation change is accepted, reviewers MUST verify all applicable items:

- [ ] The change preserves locked Product Requirements, Financial Engine, UX Foundation, and Design System decisions.
- [ ] The change serves product needs before learning goals.
- [ ] Android and macOS remain first-class; iOS and Web have not entered V1 scope without approval.
- [ ] Kotlin Multiplatform and Compose Multiplatform remain the application baseline.
- [ ] The application remains local-first with no required account, authentication, backend, or network connection.
- [ ] The topology remains a modular monolith with controlled dependencies.
- [ ] Domain rules and invariants remain in a rich, pragmatic domain model.
- [ ] Logical CQRS has not been misinterpreted as distributed infrastructure or event sourcing.
- [ ] Domain events are meaningful past-tense occurrences and event sourcing has not been introduced.
- [ ] Canonical facts, derived actual state, and future projections remain distinct.
- [ ] Financial calculations are deterministic, pure where practical, and based on an explicit acyclic dependency graph.
- [ ] Derived state and persisted read models remain rebuildable from canonical facts and deterministic rules.
- [ ] Correctness and rebuild equivalence are proven before incremental optimization is relied upon.
- [ ] Financially critical state is immediately consistent; only secondary reactions may lag.
- [ ] Secondary reactions are idempotent where applicable and cannot undermine canonical correctness.
- [ ] SQLite and SQLDelight remain behind repository/domain-facing ports.
- [ ] No Kafka, RabbitMQ, external broker, transactional outbox, or microservices have entered V1.
- [ ] Financial results are explainable from the actual deterministic calculation path.
- [ ] Money uses the domain value object, supports IDR only, and avoids floating-point representation.
- [ ] User time-zone selection is limited to `Asia/Jakarta`, `Asia/Makassar`, and `Asia/Jayapura` for V1.
- [ ] Audit history is preserved without making the system event-sourced.
- [ ] The standard security baseline is met, including input validation, least privilege, and safe handling of financial data.
- [ ] Export/import/backup/restore remain distinct from multi-device synchronization.
- [ ] No live SQLite file synchronization or Syncthing-based database synchronization is used or recommended.
- [ ] A Spring Boot sync service has not been implemented without an actual approved multi-device requirement.
- [ ] Any AI capability is outside authoritative calculations and writes, with deterministic validation and explicit user confirmation.
- [ ] Shared and platform-specific responsibilities follow Section 20 without duplicating business rules.
- [ ] Unit, integration, golden financial, state-machine, architecture fitness, and valuable property-based tests cover the change as applicable.
- [ ] No deferred detailed design has been prematurely treated as constitutionally locked.
- [ ] Significant choices have an ADR, and any constitutional or scope change has an explicitly approved Change Request.

---

**End of Architecture Constitution v1.0**
