# Developer readiness and documentation map

**Status:** Working coordination documents; non-normative. **Last inventory:** 2026-11-18. **Owner:** unassigned; project owner to assign. Review on each slice review and whenever a linked baseline changes; no formal review has yet been recorded.

Use the [repository authority hierarchy](../../README.md#documentation-authority). The frozen product baseline and financial engine govern behavior; architecture implements them. These readiness documents identify coverage and proposals and cannot change approved rules. An inventory or draft does not imply stakeholder approval or user validation.

| Artifact | Purpose and authority | Status / dependencies |
|---|---|---|
| [Product vision](../product/01-product-vision.md), [PRD](../product/02-v1-prd.md), [scope](../product/03-v1-scope.md) | Normative product intent and boundaries | Frozen; changes require reconciliation |
| [Financial engine](../requirements/04-financial-engine.md), [functional](../requirements/05-functional-requirements.md), [edge cases](../requirements/06-edge-cases.md), [NFRs](../requirements/07-non-functional-requirements.md) | Normative calculations, behavior and quality | Frozen; dependent on product baseline |
| [Conceptual domain model](../design/08-domain-model.md), [glossary](../reference/11-glossary.md) | Normative concepts and language | Frozen; dependent on product and engine |
| [Information architecture](../design/09-information-architecture.md), [flows](../design/10-user-flows.md) | Normative UX structure and flows | Frozen; subordinate to requirements |
| [Design system](../design/DESIGN.md), [screen assets](../design/assets/CHANGELOG.md) | Canonical visual foundation and illustrative screens | Existing; assets do not override behavior |
| [Analytics](../reference/12-analytics.md) | Normative metrics and privacy rules | Frozen; depends on product and NFRs |
| [Architecture constitution](../architecture/ARCHITECTURE_CONSTITUTION.md) and [strategic DDD](../architecture/strategic-ddd.md) | Normative engineering constraints and context ownership | Frozen; subordinate to product requirements |
| [Calendar tactical DDD](../architecture/financial-calendar-tactical-ddd-v1.0.md) | Normative calendar domain model | Frozen; implementation details deferred |
| [Ledger tactical baseline](../architecture/financial-accounts-ledger-tactical-ddd-draft.md) | Owner-frozen ledger model and contracts; current disposition in §30 | Frozen v1.0 with open findings; affected implementation paths remain blocked pending review and resolution |
| [Assumptions and research](assumptions-research.md) | Evidence register and research protocol | Open; no user validation claimed |
| [Traceability](traceability.md) | PRD and functional requirement coverage | Working; links to scenarios and explicitly identifies gaps |
| [Acceptance catalog](acceptance-catalog.md) | Concrete examples for future tests | Draft; subordinate to normative sources; no tests executed |
| [UX coverage](ux-coverage.md) | Flow-to-screen and state inventory | Working; review and usability evidence outstanding |
| [Ledger product decisions](ledger-product-decisions.md) | Owner-approved product clarifications and remaining review | Recorded; technical design and identified details remain open |
| [Decisions and questions](decisions.md) | Open decisions, evidence and impacts | Open; owners and approvals unassigned |
| [Delivery backlog](delivery-backlog.md) | Ordered slices, dependencies and readiness gates | Proposed; no slice approved as ready |

## Readiness rule

A slice is ready only after its owner and reviewers are assigned and approve the linked requirements, boundaries, flows, NFRs, acceptance cases, and unresolved-issue disposition. Blocking product questions require requirements reconciliation; architectural choices use the constitution's ADR process. Review all affected frozen documents before promotion. Golden tests are to be implemented and run in the application repository, and the [NFR release gates](../requirements/07-non-functional-requirements.md#14-release-gates) remain mandatory.
