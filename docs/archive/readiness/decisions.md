# Questions and decisions register

**Status:** Open working register; no new product decisions are approved here. Owner and reviewer for each item: unassigned. Record evidence, decision maker, date, affected IDs and links to the approved change/ADR before closure.

| ID | Question or decision | Impact / next action | Status |
|---|---|---|---|
| Q-01 | Who owns product approval, domain reviews and each delivery slice? | Assign named owners and reviewers before promoting any slice. | Open; all slices |
| Q-02 | Can representative users understand Unallocated, Flexible and STS, and complete explicit allocation? | Run R-01 and R-02 in [research](assumptions-research.md); reconcile any proposed behavior changes. | Open; UX validation |
| Q-03 | Do users distinguish forecast assumptions from current spendable money and complete quick entry? | Run R-03 and R-04; record observed outcomes. | Open; UX validation |
| Q-04 | What are the ledger aggregate boundaries, transaction protocol and persisted correction representation? | Review [ledger draft](../architecture/financial-accounts-ledger-tactical-ddd-draft.md); preserve atomicity and audit, then decide in tactical/persistence design. | Open; blocks ledger implementation |
| Q-05 | How are ledger corrections coordinated with allocation, budgets, income links and commitment settlements? | Define cross-context command and transaction contracts without weakening immediate consistency. | Open; blocks linked corrections |
| Q-06 | Which ledger fields, validation rules and supported operations are required for each account/transaction variant? | Derive from baseline; identify gaps requiring product clarification rather than guessing. | Open; blocks affected operations |
| Q-07 | Which flows and states have reviewed platform-specific designs? | Complete [UX coverage](ux-coverage.md) and accessibility review on both platforms. | Open; blocks affected UI |
| Q-08 | What exact export format, compatibility checks, restore replacement/merge behavior and conflict handling apply? | Produce portability design under constitution and obtain product clarification for user-visible semantics. | Open; blocks portability implementation |
| Q-09 | How will performance targets be measured on representative devices and datasets? | Specify device profiles, workloads and measurement method before launch; NFR-PERF-004 explicitly requires methodology. | Open; blocks performance sign-off |
| Q-10 | Are there requirements or contracts missing from the first-pass traceability and scenario catalog? | Inspect per-ID coverage and missing fields; add scenarios, review mappings and reconcile any contradiction. | Open; blocks slice approval |
| Q-11 | Which quantitative success targets and measurement windows are appropriate? | Select after research and establish baselines, using privacy constraints in analytics. | Open; product decision |

## Recorded baseline decisions

The [strategic DDD decisions](../architecture/strategic-ddd.md#16-resolved-review-clarifications) and [calendar decision register](../architecture/financial-calendar-tactical-ddd-v1.0.md#19-decision-register--td-cal-001-through-td-cal-048) remain authoritative. This register does not duplicate or reopen them. To close an item, link the decision and supporting evidence, identify approver/date, and update affected indexes, traceability and backlog; product behavior changes follow README change control.

## Follow-up to owner-approved CR-LDG-001

The product owner accepted the choices in [CR-LDG-001](ledger-product-decisions.md). Q-04–06 remain open for the technical and product details not decided there. The owner approved the product behavior below on 2026-11-14. Concrete integration, validation and exceptional lifecycle details still require design review; approval does not freeze the tactical model:

| ID | Remaining question | Status |
|---|---|---|
| Q-12 | Unlinked refund reporting and later linking | Owner-approved: separate from income/expense, no automatic budget impact; confirmed later link recalculates attributable budget. Technical review pending. |
| Q-13 | Adjustment reporting and forecast | Owner-approved: separate report item; forecast begins from corrected actual balance. Capture contract remains technical design. |
| Q-14 | Pre-floor transactions and historical reports | Owner-approved: show fact/account effect; mark affected cycle-based reports incomplete or unavailable. Review exact report implementation. |
| Q-15 | Edit/void optional transfer fee | Owner-approved: review each effect; suggest void both, permit retaining charged fee; atomic confirmed change. Technical review pending. |
| Q-16 | Expected Income after unlink | Owner-approved: EXPECTED before expected time, MISSED after; unsafe intervening lifecycle changes require guided resolution with no partial save. Exceptional resolution contract remains open. |

## Follow-up to owner-approved CR-LDG-002

See [CR-LDG-002](ledger-product-decisions.md) for accepted refund limits, guided account correction, unsupported positive card credit and asset opening snapshots. The owner subsequently selected explicit before/after ordering for date-only transactions on the opening snapshot date and no partial recording of a card refund exceeding outstanding. [CR-LDG-003](ledger-product-decisions.md) records the approved transaction correction and payment-conflict choices. [CR-LDG-004](ledger-product-decisions.md) records snapshot edit preview and payment-time/later-history validation. [CR-LDG-005](ledger-product-decisions.md) records the user ordering choice for financially significant same-day date-only events and the unknown-debt guidance. Outstanding design questions include the historical starting-information workflow, liability opening-state semantics, changes that invalidate an ordering choice and implementation of correction consistency. No unapproved outcome may be inferred.

## Initial Ledger technical audit

See [Ledger technical audit](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#22-technical-audit--initial-pass) for findings AUD-LDG-001–009. Q-04–06 remain open; a first-pass document review is not independent approval, execution of tests or authorization to freeze. AUD-LDG-006 flags wording about unlinked actual income for reconciliation; it does not reopen the approved optional-link behavior.

## Historical balance and order revalidation

[CR-LDG-006](ledger-product-decisions.md) records the approved guided earlier-debt entry and ordering revalidation. [CR-LDG-007](ledger-product-decisions.md) approves the historical-checkpoint relationship, discrepancy handling and indeterminate-history distinction. Precise comparison intervals, ordering, checkpoint persistence, concurrency and revalidation still require technical review before implementation.

## Consistency contract review

The [Ledger consistency proposal](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#27-consistency-contracts--technical-review-proposal) expands checkpoint outcomes, atomic mutations, stale-preview detection, refund concurrency, reads and recovery. It is a draft technical proposal; AUD-LDG-001–004 and AUD-LDG-008 remain open until reviewed and verified. Same-instant ordering and liability opening-state completeness are not resolved by this proposal.

## Equal-time ordering — owner-selected recommendation

[CR-LDG-008](ledger-product-decisions.md) records the owner-selected equal-time behavior for events and balance boundaries. The order is explicit when it affects outcomes; unknown required order is indeterminate. Concrete consistency, conflict and revalidation contracts still need verification; this product decision alone does not freeze the Ledger model. `RULE-STATE-001` wording was reconciled to preserve the already approved ability to post unlinked actual income (AUD-LDG-006).

## Consolidated Ledger review candidate — 2026-11-18

[Ledger §§29–30](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#29-consolidated-tactical-model--review-candidate) consolidate candidate aggregate, operation, query and consistency contracts and give current finding dispositions. [CR-LDG-008](ledger-product-decisions.md) records owner-selected equal-time behavior; [CR-LDG-009](ledger-product-decisions.md) records a same-account-transfer recommendation selected under the owner's instruction to review decisions at the end. Q-04–06 remain review items rather than claimed resolved. See the current disposition rather than treating the dated initial-audit table as live status.

## Owner freeze — 2026-11-18

The product owner explicitly froze the [Ledger tactical baseline v1.0](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#32-owner-freeze-record) with open findings. This is not an independent audit or evidence of passing tests. Q-04–06 and the outstanding findings in §30 remain open and block affected implementation paths until resolved; product behavior changes require reconciliation/change control. Income Management is next on the tactical roadmap, alongside resolving Ledger findings.
