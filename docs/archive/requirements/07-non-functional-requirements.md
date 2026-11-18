# V1 Non-Functional Requirements

**Document ID:** NFR-V1  
**Status:** APPROVED / FROZEN — Normative  
**Scope:** Quality, security, reliability, privacy, performance, accessibility, observability, maintainability.

## 1. Principles

Because V1 produces financial guidance, correctness and explainability take precedence over decorative responsiveness. A stale or silently inconsistent STS is worse than a temporarily unavailable calculation.

## 2. Financial Correctness & Determinism

### `NFR-COR-001` Deterministic calculation
Given identical canonical data, configuration, timezone, and evaluation instant, the financial engine MUST return byte-equivalent numeric results and equivalent event ordering.

### `NFR-COR-002` Integer monetary arithmetic
Canonical monetary amounts MUST use integer minor units. Floating-point arithmetic MUST NOT be used for canonical money calculations.

### `NFR-COR-003` Explicit rounding
All divisions requiring rounding MUST use the rule explicitly specified by the Financial Engine; implementation-default rounding MUST NOT determine financial behavior.

### `NFR-COR-004` Derived-data reproducibility
STS, Daily STS, Funding Deficit, Available, Unallocated, forecast paths, and similar derived outputs MUST be reproducible from canonical records.

### `NFR-COR-005` Golden financial fixtures
CI MUST execute canonical numerical scenarios from `04-financial-engine.md` and `06-edge-cases.md`. A release MUST NOT pass if exact expected minor-unit results differ.

## 3. Atomicity & Data Integrity

### `NFR-DATA-001`
Multi-record financial mutations MUST be transactional/atomic.

### `NFR-DATA-002`
Referential constraints MUST prevent orphaned settlement, reconciliation, reservation, statement, and installment relationships.

### `NFR-DATA-003`
Financially relevant destructive actions MUST preserve audit semantics through void/cancel/supersede where required.

### `NFR-DATA-004`
Concurrent writes MUST not allow committed allocation oversubscription or duplicate reconciliation/settlement.

### `NFR-DATA-005`
Every canonical record SHOULD carry stable identifier, creation timestamp, update timestamp, and status/provenance fields appropriate to its entity.

## 4. Security

### `NFR-SEC-001` Transport security
All client-server traffic containing user or financial data MUST use TLS 1.2+; TLS 1.3 SHOULD be preferred where supported.

### `NFR-SEC-002` Encryption at rest
Sensitive user and financial data MUST be encrypted at rest using platform/provider mechanisms appropriate to the deployed architecture.

### `NFR-SEC-003` Authentication
Authenticated deployments MUST use established secure authentication/session mechanisms. Passwords, if directly handled, MUST use an industry-standard adaptive password hashing algorithm; plaintext passwords MUST never be stored.

### `NFR-SEC-004` Authorization
Every server-side access to user financial records MUST enforce ownership/authorization independent of client-side filtering.

### `NFR-SEC-005` Secrets
Secrets, API keys, signing keys, and database credentials MUST NOT be committed to source control or exposed to clients unless specifically designed as public client credentials.

### `NFR-SEC-006` Input validation
All external inputs MUST be validated server-side or at the trusted persistence boundary even if already validated by the client.

### `NFR-SEC-007` Logging
Application logs MUST NOT contain authentication secrets, full sensitive credentials, or unnecessary raw financial payloads.

### `NFR-SEC-008` Dependency security
Production dependencies SHOULD be continuously scanned for known vulnerabilities and critical/high exploitable findings MUST be addressed before release or explicitly risk-accepted.

## 5. Privacy & Data Minimization

### `NFR-PRIV-001`
Collect only data necessary for V1 functionality, security, support, and explicitly defined analytics.

### `NFR-PRIV-002`
Analytics MUST NOT send transaction descriptions, account names, obligation descriptions, income source names, exact account balances, exact transaction amounts, or other free-text financial content by default.

### `NFR-PRIV-003`
Analytics identifiers SHOULD be pseudonymous and MUST NOT be used as authorization identifiers.

### `NFR-PRIV-004`
Where deletion/export capabilities are offered, behavior MUST be documented and must preserve only data that is legitimately required to remain.

### `NFR-PRIV-005`
Production financial data MUST NOT be copied into non-production environments without approved anonymization/sanitization.

### `NFR-PRIV-006`
The implementation MUST support applicable privacy-law obligations for the markets in which the product is launched; legal compliance specifics are deployment obligations and MUST NOT be inferred solely from this product specification.

## 6. Performance

Targets apply under normal supported V1 dataset sizes and healthy infrastructure.

### `NFR-PERF-001`
Local/client validation feedback SHOULD occur within 100 ms for ordinary inputs.

### `NFR-PERF-002`
Financial engine recalculation for a typical user dataset SHOULD complete within 200 ms at p95 on the execution tier responsible for calculation, excluding network latency.

### `NFR-PERF-003`
Interactive API reads/writes SHOULD complete within 500 ms server processing time at p95, excluding client network latency and third-party dependencies.

### `NFR-PERF-004`
Primary dashboard SHOULD become meaningfully usable within 2.5 seconds at p75 under representative supported-device/network conditions; exact measurement methodology MUST be defined before launch.

### `NFR-PERF-005`
A V1 account SHOULD support at least 10,000 transactions, 1,000 obligations, and 1,000 expected-income/forecast events without violating calculation correctness; performance degradation beyond typical use MUST be graceful.

## 7. Reliability & Recovery

### `NFR-REL-001`
A failed financial mutation MUST leave canonical state equivalent to the state before the attempted mutation.

### `NFR-REL-002`
If derived calculation fails, the product MUST NOT present a newly calculated STS as authoritative. It SHOULD show the last known result only if clearly marked stale with its evaluation time.

### `NFR-REL-003`
Server-backed deployments MUST maintain automated backups of canonical persistent data with documented restore procedures.

### `NFR-REL-004`
Before production launch, target RPO and RTO MUST be explicitly chosen based on deployment architecture. Recommended initial product target: RPO ≤ 24 hours and RTO ≤ 4 hours unless stronger requirements are justified.

### `NFR-REL-005`
Restore procedures MUST be tested periodically; existence of a backup without a tested restoration path is insufficient.

## 8. Offline and Synchronization Boundary

### `NFR-SYNC-001`
V1 MUST explicitly choose one source-of-truth model per deployment: local-first, server-authoritative, or another documented model. Implementations MUST NOT mix authority implicitly.

### `NFR-SYNC-002`
If offline mutation is supported, conflict resolution for financial canonical records MUST be deterministic and MUST NOT use silent last-write-wins where it could violate financial invariants.

### `NFR-SYNC-003`
If offline mutation is not supported, the UI MUST clearly distinguish unsaved local input from committed financial state.

## 9. Accessibility

### `NFR-A11Y-001`
User-facing web interfaces SHOULD target WCAG 2.2 AA.

### `NFR-A11Y-002`
Financial state MUST NOT be communicated by color alone.

### `NFR-A11Y-003`
Interactive controls MUST have accessible names, keyboard operability where applicable, and meaningful focus order.

### `NFR-A11Y-004`
Formatted currency and dates SHOULD remain understandable to assistive technologies.

## 10. Localization

### `NFR-I18N-001`
Currency/date/number presentation MUST use locale-aware formatting independent from canonical storage.

### `NFR-I18N-002`
Financial calculations MUST use configured financial timezone, not device timezone implicitly.

### `NFR-I18N-003`
User-visible copy SHOULD be externalizable for localization. Financial rule identifiers and canonical enum values MUST remain stable across languages.

## 11. Observability

### `NFR-OBS-001`
The system MUST expose operational telemetry sufficient to detect failed financial mutations, engine calculation failures, latency regressions, and persistence errors without logging unnecessary financial content.

### `NFR-OBS-002`
Every server-side financial mutation SHOULD have a correlation/request identifier usable for debugging.

### `NFR-OBS-003`
Alerts SHOULD exist for sustained error-rate increases and financial-engine failures before production launch.

### `NFR-OBS-004`
Analytics events MUST be distinguishable from operational logs; disabling/failing product analytics MUST NOT impair financial functionality.

## 12. Maintainability

### `NFR-MNT-001`
Financial-engine logic MUST be isolated from presentation logic sufficiently to execute deterministic unit/domain tests without UI.

### `NFR-MNT-002`
Canonical financial formulas MUST have one authoritative implementation path per platform/runtime; duplicated formulas across UI components are prohibited.

### `NFR-MNT-003`
Every normative `RULE-*` and high-risk `EC-*` SHOULD map to automated tests. Golden fixtures MUST cover: newly available money remaining Unallocated; STS excluding Unallocated and all Expected Income; strict current-cycle STS boundary; Base vs Scenario Forecast; Funding Dependency and its removal; Budget > Flexible warning without reservation; Day 1/2 projection suppression and Day 3 eligibility; Financial Cycle weekend adjustment; budget reset/template; and credit/BNPL no-double-counting.

### `NFR-MNT-004`
Schema migrations affecting canonical financial records MUST be versioned, reversible where practical, and tested against representative prior-state fixtures.

### `NFR-MNT-005`
Provider-specific names/rules MUST be data/configuration driven where V1 supports provider customization; they MUST NOT leak into generic financial-engine invariants.

## 13. Compatibility

### `NFR-COMP-001`
Supported OS/browser versions MUST be explicitly declared before release and tested in CI/device coverage.

### `NFR-COMP-002`
Unsupported clients MUST fail safely and MUST NOT silently calculate financial guidance using incompatible rule versions.

### `NFR-COMP-003`
If client and server both execute financial logic, a financial-engine/schema version MUST be available for compatibility checks.

## 14. Release Gates

A V1 production release MUST NOT proceed when:
- canonical golden financial tests fail;
- known defect can silently overstate STS;
- allocation shortfall can occur without explicit shortfall semantics and STS degradation;
- authorization permits cross-user financial-data access;
- migration can cause unexplained monetary loss/duplication;
- critical financial mutation is non-atomic;
- security-critical known issue is unresolved without explicit risk acceptance.

## 15. Priority

For conflicting implementation tradeoffs, V1 priority is:

```text
Financial Correctness
> Data Integrity
> Security / Privacy
> Explainability
> Reliability
> Accessibility
> Performance
> Visual Polish
```

This ordering does not permit knowingly unacceptable security/privacy behavior; it resolves ordinary product-engineering tradeoffs within acceptable baselines.
