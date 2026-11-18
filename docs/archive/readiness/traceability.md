# Requirements traceability matrix

**Status:** First-pass navigation and coverage inventory, not a reviewed verification matrix. Every PRD and individually numbered functional requirement is listed. References in a row identify relevant *families*, not proven one-to-one coverage. **Gap/review required** means no claim that the entire requirement has an acceptance test, flow, or edge case. Unnumbered atomicity requirements in functional section 13 also require review. See [Q-10](decisions.md).

Sources: [PRD](../product/02-v1-prd.md) · [functional requirements](../requirements/05-functional-requirements.md) · [financial engine](../requirements/04-financial-engine.md) · [flows](../design/10-user-flows.md) · [edge cases](../requirements/06-edge-cases.md) · [NFR](../requirements/07-non-functional-requirements.md) · [acceptance catalog](acceptance-catalog.md). Ranges denote existing families, not a guarantee every suffix exists. NFRs listed are representative, not exhaustive.

## Functional requirements

| Requirement | Related PRD | Engine family | Flow | Edge family | NFR example | Scenario | Coverage |
|---|---|---|---|---|---|---|---|
| FR-ONB-001 Short onboarding | PRD-ONB-001 | RULE-ALLOC-101A | UF-ONB-001 | EC-ALLOC-NEW-001 | NFR-I18N-001 | AC-01 | Review required; per-clause gaps possible |
| FR-CYCLE-001 Configure cycle | PRD-CYCLE-001 | RULE-CYCLE-101–106 | UF-CYCLE-001 | EC-CYCLE-001–009 | NFR-COR-001 | AC-08 | Review required; per-clause gaps possible |
| FR-ACC-001 Asset accounts | PRD-TRX-001 | RULE-BAL-001–002 | gap | gap | NFR-DATA-001 | AC-06 | Review required; per-clause gaps possible |
| FR-ACC-002 Liability accounts | PRD-TRX-001 | RULE-BAL-001–002 | gap | gap | NFR-DATA-001 | AC-06 | Review required; per-clause gaps possible |
| FR-ACC-003 Archive/history | PRD-TRX-001 | RULE-BAL-001–002 | gap | gap | NFR-DATA-001 | AC-06 | Review required; per-clause gaps possible |
| FR-TRX-001 Quick expense | PRD-TRX-001; PRD-TRX-002 | RULE-BAL-001–002 | UF-TRX-001; UF-TRX-002; UF-EDIT-001 | EC-TRX-001–004 | NFR-DATA-001 | AC-02–07 | Review required; per-clause gaps possible |
| FR-TRX-002 Income | PRD-TRX-001; PRD-TRX-002 | RULE-BAL-001–002 | UF-TRX-001; UF-TRX-002; UF-EDIT-001 | EC-TRX-001–004 | NFR-DATA-001 | AC-02–07 | Review required; per-clause gaps possible |
| FR-TRX-003 Expense | PRD-TRX-001; PRD-TRX-002 | RULE-BAL-001–002 | UF-TRX-001; UF-TRX-002; UF-EDIT-001 | EC-TRX-001–004 | NFR-DATA-001 | AC-02–07 | Review required; per-clause gaps possible |
| FR-TRX-004 Transfer | PRD-TRX-001; PRD-TRX-002 | RULE-BAL-001–002 | UF-TRX-001; UF-TRX-002; UF-EDIT-001 | EC-TRX-001–004 | NFR-DATA-001 | AC-02–07 | Review required; per-clause gaps possible |
| FR-TRX-005 Edit/void/correction | PRD-TRX-001; PRD-TRX-002 | RULE-BAL-001–002 | UF-TRX-001; UF-TRX-002; UF-EDIT-001 | EC-TRX-001–004 | NFR-DATA-001 | AC-02–07 | Review required; per-clause gaps possible |
| FR-ALLOC-001  | PRD-ALLOC-001 | RULE-ALLOC-101–105 | UF-ALLOC-001 | EC-ALLOC-NEW-001–005 | NFR-DATA-001 | AC-01; AC-05 | Review required; per-clause gaps possible |
| FR-ALLOC-002  | PRD-ALLOC-001 | RULE-ALLOC-101–105 | UF-ALLOC-001 | EC-ALLOC-NEW-001–005 | NFR-DATA-001 | AC-01; AC-05 | Review required; per-clause gaps possible |
| FR-ALLOC-003  | PRD-ALLOC-001 | RULE-ALLOC-101–105 | UF-ALLOC-001 | EC-ALLOC-NEW-001–005 | NFR-DATA-001 | AC-01; AC-05 | Review required; per-clause gaps possible |
| FR-ALLOC-004  | PRD-ALLOC-001 | RULE-ALLOC-101–105 | UF-ALLOC-001 | EC-ALLOC-NEW-001–005 | NFR-DATA-001 | AC-01; AC-05 | Review required; per-clause gaps possible |
| FR-BUD-001 Create cycle budget | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-002 Budget health | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-003 Warning thresholds | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-004 Spending pace | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-005 Projection | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-006 Cycle close | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-007 Previous-cycle template | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-BUD-008 Refund/category correction | PRD-BUD-001; PRD-BUD-002 | RULE-BUD-101–109; RULE-PROJ-BUD-101–102 | UF-BUD-001–003 | EC-BUD-001–010; EC-PROJ-001–006 | NFR-COR-001 | AC-10–12 | Review required; per-clause gaps possible |
| FR-OBL-001  | PRD-OBL-001 | RULE-OBL-101–104 | UF-OBL-001 | EC-OBL-001–004 | NFR-COR-001 | AC-09; AC-15 | Review required; per-clause gaps possible |
| FR-OBL-002  | PRD-OBL-001 | RULE-OBL-101–104 | UF-OBL-001 | EC-OBL-001–004 | NFR-COR-001 | AC-09; AC-15 | Review required; per-clause gaps possible |
| FR-OBL-003  | PRD-OBL-001 | RULE-OBL-101–104 | UF-OBL-001 | EC-OBL-001–004 | NFR-COR-001 | AC-09; AC-15 | Review required; per-clause gaps possible |
| FR-REC-001 Recurring obligations | PRD-OBL-001 | RULE-REC-101–102 | UF-OBL-001 | EC-REC-001–002 | NFR-COR-001 | AC-18 | Review required; per-clause gaps possible |
| FR-REC-002 Staged amounts | PRD-OBL-001 | RULE-REC-101–102 | UF-OBL-001 | EC-REC-001–002 | NFR-COR-001 | AC-18 | Review required; per-clause gaps possible |
| FR-INC-001  | PRD-INC-001 | RULE-FCST-102–103A | UF-INC-001; UF-INC-002 | EC-INC-001–006 | NFR-DATA-001 | AC-03; AC-13; AC-14 | Review required; per-clause gaps possible |
| FR-INC-002  | PRD-INC-001 | RULE-FCST-102–103A | UF-INC-001; UF-INC-002 | EC-INC-001–006 | NFR-DATA-001 | AC-03; AC-13; AC-14 | Review required; per-clause gaps possible |
| FR-INC-003  | PRD-INC-001 | RULE-FCST-102–103A | UF-INC-001; UF-INC-002 | EC-INC-001–006 | NFR-DATA-001 | AC-03; AC-13; AC-14 | Review required; per-clause gaps possible |
| FR-INC-004  | PRD-INC-001 | RULE-FCST-102–103A | UF-INC-001; UF-INC-002 | EC-INC-001–006 | NFR-DATA-001 | AC-03; AC-13; AC-14 | Review required; per-clause gaps possible |
| FR-INC-005 Late/missed expected income | PRD-INC-001 | RULE-FCST-102–103A | UF-INC-001; UF-INC-002 | EC-INC-001–006 | NFR-DATA-001 | AC-03; AC-13; AC-14 | Review required; per-clause gaps possible |
| FR-STS-001  | PRD-STS-001 | RULE-STS-101–103; RULE-DSTS-101–103 | UF-STS-001 | EC-STS-001–004; EC-STS-BND-001–002 | NFR-COR-001 | AC-01; AC-05; AC-09 | Review required; per-clause gaps possible |
| FR-STS-002  | PRD-STS-001 | RULE-STS-101–103; RULE-DSTS-101–103 | UF-STS-001 | EC-STS-001–004; EC-STS-BND-001–002 | NFR-COR-001 | AC-01; AC-05; AC-09 | Review required; per-clause gaps possible |
| FR-STS-003  | PRD-STS-001 | RULE-STS-101–103; RULE-DSTS-101–103 | UF-STS-001 | EC-STS-001–004; EC-STS-BND-001–002 | NFR-COR-001 | AC-01; AC-05; AC-09 | Review required; per-clause gaps possible |
| FR-FCST-001  | PRD-FCST-001 | RULE-FCST-101–106 | UF-FCST-001; UF-FCST-002 | EC-FDEP-001–005 | NFR-COR-001 | AC-13; AC-16 | Review required; per-clause gaps possible |
| FR-FCST-002  | PRD-FCST-001 | RULE-FCST-101–106 | UF-FCST-001; UF-FCST-002 | EC-FDEP-001–005 | NFR-COR-001 | AC-13; AC-16 | Review required; per-clause gaps possible |
| FR-FCST-003  | PRD-FCST-001 | RULE-FCST-101–106 | UF-FCST-001; UF-FCST-002 | EC-FDEP-001–005 | NFR-COR-001 | AC-13; AC-16 | Review required; per-clause gaps possible |
| FR-FCST-004 Funding Dependency | PRD-FCST-001 | RULE-FCST-101–106 | UF-FCST-001; UF-FCST-002 | EC-FDEP-001–005 | NFR-COR-001 | AC-13; AC-16 | Review required; per-clause gaps possible |
| FR-CRD-001  | PRD-TRX-001; PRD-OBL-001 | RULE-CC-101–104 | UF-CC-001 | EC-CC-001–003 | NFR-DATA-001 | AC-04 | Review required; per-clause gaps possible |
| FR-CRD-002  | PRD-TRX-001; PRD-OBL-001 | RULE-CC-101–104 | UF-CC-001 | EC-CC-001–003 | NFR-DATA-001 | AC-04 | Review required; per-clause gaps possible |
| FR-CRD-003  | PRD-TRX-001; PRD-OBL-001 | RULE-CC-101–104 | UF-CC-001 | EC-CC-001–003 | NFR-DATA-001 | AC-04 | Review required; per-clause gaps possible |
| FR-INST-001  | PRD-OBL-001 | RULE-INST-102–103 | UF-CC-001; UF-BNPL-001 | EC-INST-001–002 | NFR-DATA-001 | gap | Review required; per-clause gaps possible |
| FR-BNPL-001  | PRD-TRX-001; PRD-OBL-001 | RULE-INST-101–103 | UF-BNPL-001 | EC-BNPL-001 | NFR-DATA-001 | AC-04 (credit only); gap | Review required; per-clause gaps possible |
| FR-ADV-001  | PRD-ADV-001 | RULE-ADV-101–105 | UF-ADV-001 | EC-ADV-001 | NFR-COR-001 | AC-17 | Review required; per-clause gaps possible |
| FR-ADV-002  | PRD-ADV-001 | RULE-ADV-101–105 | UF-ADV-001 | EC-ADV-001 | NFR-COR-001 | AC-17 | Review required; per-clause gaps possible |
| FR-ADV-003  | PRD-ADV-001 | RULE-ADV-101–105 | UF-ADV-001 | EC-ADV-001 | NFR-COR-001 | AC-17 | Review required; per-clause gaps possible |
| FR-NAV-001  | scope navigation | not applicable | IA sections 1–6 | gap | NFR-A11Y-001 | gap | Review required; per-clause gaps possible |
| FR-HOME-001  | PRD-STS-001; PRD-ADV-001 | RULE-STS-101; RULE-ADV-101–105 | UF-STS-001; UF-ADV-001 | EC-STS-001–004 | NFR-A11Y-001 | AC-09; AC-17 | Review required; per-clause gaps possible |


| FR-LDG-001 Archive and account class | CR-LDG-001 | RULE-AVAILABLE-001 | UF-LDG-001 | EC-LDG-001 | NFR-DATA-003 | AC-19 | Review required; guided correction detail open |
| FR-LDG-002 Adjustment and negative tracked balance | CR-LDG-001 | RULE-LDG-001; RULE-LDG-003 | UF-LDG-002 | EC-LDG-002 | NFR-DATA-003 | AC-20 | Review approved reporting and forecast behavior |
| FR-LDG-003 Correct and void | PRD-TRX-001; CR-LDG-001 | RULE-LDG-003 | UF-EDIT-001; UF-LDG-005 | EC-LDG-008 | NFR-DATA-001 | AC-07; AC-14 | Review exceptional linked-correction handling |
| FR-LDG-004 Liability payment limit | CR-LDG-001 | RULE-LDG-002 | UF-CC-001 | EC-LDG-003 | NFR-REL-001 | AC-21 | Review required |
| FR-LDG-005 Refunds | PRD-TRX-001; CR-LDG-001 | RULE-LDG-003 | UF-LDG-004 | EC-LDG-004 | NFR-COR-001 | AC-07 (related) | Review confirmed later-link integration |
| FR-LDG-006 Category lifecycle | CR-LDG-001 | RULE-BUD-101 | gap | EC-LDG-007 | NFR-DATA-003 | gap | Review and scenario gap |
| FR-LDG-007 Before calendar history | CR-LDG-001 | RULE-LDG-004 | UF-LDG-006 | EC-LDG-006 | NFR-COR-001 | AC-23 | Review affected report completeness |
| FR-LDG-008 Transfer fee | CR-LDG-001 | RULE-LDG-003 | UF-LDG-003 | EC-LDG-005 | NFR-DATA-001 | AC-22 | Review atomic fee correction and void |


| FR-LDG-009 Refund limits and corrections | CR-LDG-002 | RULE-LDG-003 | UF-LDG-008 | EC-LDG-012; EC-LDG-013 | NFR-DATA-001 | AC-27 | Review concurrent linked refunds |
| FR-LDG-010 Guided account-class correction | CR-LDG-002 | RULE-BAL-001–002 | UF-LDG-007 | gap | NFR-DATA-003 | gap | Review guided correction coverage |
| FR-LDG-011 Credit refund beyond liability | CR-LDG-002 | RULE-BAL-002 | gap | EC-LDG-014 | NFR-COR-001 | gap | Owner-approved full-refund limitation; technical review pending |
| FR-LDG-012 Asset opening snapshot | CR-LDG-002 | RULE-BAL-001 | UF-LDG-009 | EC-LDG-015 | NFR-COR-001 | AC-28 | Owner-approved snapshot edits and transaction correction; technical review pending |

| FR-LDG-013 Correction invalidating liability payment | CR-LDG-003 | RULE-LDG-002 | UF-LDG-013 | EC-LDG-019 | NFR-DATA-001 | AC-32 | Review general historical liability ordering |

| FR-LDG-014 Backdated credit-card payment | CR-LDG-004 | RULE-LDG-002 | UF-LDG-015 | EC-LDG-021; EC-LDG-022; EC-LDG-023 | NFR-DATA-001 | AC-34; AC-35; AC-36 | Starting-information workflow and ordering invalidation review pending |

| FR-LDG-015 Guided historical card debt | CR-LDG-006; CR-LDG-007 | RULE-LDG-002; RULE-LDG-005 | UF-LDG-016 | EC-LDG-024; EC-LDG-026–027 | NFR-DATA-001; NFR-DATA-003 | AC-37; AC-39–40 | Technical interval and concurrency review pending |
| FR-LDG-016 Revalidate saved card order | CR-LDG-006 | RULE-LDG-002 | UF-LDG-017 | EC-LDG-025 | NFR-DATA-001 | AC-38 | Detailed applicability/conflict contract to review |

| FR-LDG-017 Equal-time ordering | CR-LDG-008 | RULE-LDG-006 | UF-LDG-018 | EC-LDG-028–030 | NFR-COR-001; NFR-DATA-001 | AC-41–43 | Owner-selected; technical verification pending |

| FR-LDG-018 Same-account transfer | CR-LDG-009 | RULE-BAL-001 | UF-TRX-002 | EC-LDG-031 | NFR-DATA-001 | AC-44 | Owner review of delegated selection pending |

## Product requirements

| PRD requirement | Related functional requirements | Trace status |
|---|---|---|
| PRD-TRX-001 Transactions | FR-TRX-002–005; FR-CRD-001–003; FR-BNPL-001 | Review linked functional rows and scenario gaps |
| PRD-TRX-002 Quick expense capture | FR-TRX-001 | Review linked functional rows and scenario gaps |
| PRD-BUD-001 Spending budgets | FR-BUD-001–005 | Review linked functional rows and scenario gaps |
| PRD-BUD-002 Budget reset/template | FR-BUD-006–007 | Review linked functional rows and scenario gaps |
| PRD-ALLOC-001 Money allocation | FR-ALLOC-001–004; FR-TRX-002 | Review linked functional rows and scenario gaps |
| PRD-INC-001 Expected income | FR-INC-001–005 | Review linked functional rows and scenario gaps |
| PRD-STS-001 Safe to Spend | FR-STS-001–003 | Review linked functional rows and scenario gaps |
| PRD-FCST-001 Forecast | FR-FCST-001–004; FR-INC-004 | Review linked functional rows and scenario gaps |
| PRD-CYCLE-001 Financial Cycle | FR-CYCLE-001 | Review linked functional rows and scenario gaps |
| PRD-OBL-001 Obligations | FR-OBL-001–003; FR-REC-001–002; FR-CRD-002; FR-INST-001 | Review linked functional rows and scenario gaps |
| PRD-ADV-001 Deterministic insights | FR-ADV-001–003 | Review linked functional rows and scenario gaps |
| PRD-ONB-001 Short onboarding | FR-ONB-001 | Review linked functional rows and scenario gaps |

## Known gaps and review procedure

- No dedicated acceptance cases yet for several account variants, warning and pace boundaries, budget corrections, credit statements, installment schedules, navigation, accessibility, security and recovery. A related group case does not close these gaps.
- Each reviewer must verify individual IDs, every behavioral clause and cross-cutting NFR applicability against source text, replace group-level references with precise ones, add or link missing cases and record review evidence. Do not mark a row complete solely because an ID appears here.
- Investigate any conflicting requirements via [decisions](decisions.md); use frozen-baseline change control. No contradiction is asserted without source-level review.
- Product requirements are mapped via functional rows above; their engine, flow, edge, NFR and scenario links are reached through those rows. Section 13 atomicity, analytics and NFR release gates are cross-cutting and must be checked per slice.
