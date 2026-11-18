# Financial Accounts & Ledger — Tactical DDD Frozen Baseline v1.0

**Status:** FROZEN v1.0 — OWNER-APPROVED WITH OPEN FINDINGS  
**Bounded context:** Financial Accounts & Ledger  
**Effective date:** 2026-11-18 (Asia/Jakarta)  
**Approval:** Product owner explicitly requested freezing in the project conversation.  
**Implementation readiness:** Limited; unresolved findings in §30 block affected work until resolved. Independent review and executable tests have not been completed.

The owner freezes the consolidated model and contracts in **§29** as the current tactical baseline, subject to the higher-authority sources in §1. The owner-approved product decisions in CR-LDG-001–009 remain governed by their reconciled product and financial requirements. Earlier candidate descriptions and audit tables are retained as historical review material, not competing current specifications. Where earlier text says draft, proposal, unapproved or not frozen, it records its status at that point in the review; the current status and disposition are this header, §29, §30 and §32. The filename retains `draft` to preserve existing links; it does not indicate the current freeze status.

Freezing records the owner's approval of the current baseline; it does **not** declare unresolved financial or cross-context behavior complete, waive the constitution's change-control rules, or imply a passed audit or test suite. Implementers MUST NOT guess at remaining behavior. Record and resolve findings and reconcile affected sources before implementing an affected path.

## 1. Authority and review approach

The governing sources are the [product requirements](../product/02-v1-prd.md), [scope](../product/03-v1-scope.md), [financial engine](../requirements/04-financial-engine.md), [functional requirements](../requirements/05-functional-requirements.md), [edge cases](../requirements/06-edge-cases.md), [NFRs](../requirements/07-non-functional-requirements.md), [conceptual model](../design/08-domain-model.md), [architecture constitution](ARCHITECTURE_CONSTITUTION.md), [strategic DDD](strategic-ddd.md), and [Financial Calendar tactical DDD](financial-calendar-tactical-ddd-v1.0.md). The [glossary](../reference/11-glossary.md) governs terminology.

If a proposal conflicts with a higher source, the source prevails until a change is approved. Questions that affect user-visible financial outcomes require product clarification and reconciliation; implementation choices are reviewed under architecture governance. See [open questions](../readiness/decisions.md).

## 2. Responsibility and boundary

The context answers: **What accounts exist, and what actually happened financially?** It owns actual account and transaction facts, transaction categories, account state and liability outstanding. A facility such as a credit card or PayLater account is a liability, not cash. Provider names and terms are configurable data, not hardcoded business logic.

| Context | Owns | Ledger relationship |
|---|---|---|
| Accounts & Ledger | Accounts, categories, actual economic facts and account/liability state | Source of actual activity and balances. |
| Income Management | Expected-income occurrences and their lifecycle | An actual Income may explicitly link an occurrence; similarity never creates that link. |
| Money Allocation | Flexible, Reserved, Locked and Unallocated intent | Consumes changes in actual eligible money; Ledger does not decide allocation destinations. |
| Spending Planning | Cycle budgets, mappings and budget health | Consumes eligible actual expense/refund facts; it does not own Ledger transactions. |
| Financial Commitments | Obligations, schedules and settlement semantics | Coordinates with actual liability purchases and payments; an obligation is not itself a Ledger transaction. |
| Financial Calendar | Cycle configuration and boundary resolution | Sole authority for cycle boundaries; Ledger must not reimplement them. |
| Financial Intelligence | STS, forecasting and derived guidance | Reads authoritative actual facts and other inputs; forecasts never mutate Ledger facts. |

Exact integration contracts are not yet approved. Strategic DDD requires financially critical consistency at successful completion; a domain event alone cannot provide it.

## 3. Candidate tactical model

The following is a decomposition for discussion, **not a frozen aggregate map**.

| Candidate concept | Responsibility | Identity / lifetime | Open design work |
|---|---|---|---|
| Account | Classify a tracked asset or liability; retain opening state, eligibility and archive status. | Stable account identity; history survives archive. | Aggregate boundary; constraints on changes and opening-state correction. |
| Transaction | Represent one realized economic event, its financial time, amount, kind and affected accounts. | Stable transaction identity, with posted/voided semantics and audit history. | Aggregate boundary, correction representation, concurrency. |
| Transaction leg / account effect | Express an event's effect on an account; a transfer or liability payment has multiple effects. | Candidate internal representation; do not assume a persisted leg schema. | Identity, representation and whether derived or stored. |
| Category | Classify expenses/fees for budget mapping. | Stable user/system-defined concept. | Lifecycle, edit/archive constraints and historical behavior. |
| Money and financial time | Represent currency/minor units and effective financial time consistently. | Value semantics. | Shared implementation and exact input contracts. |
| Audit/provenance | Explain creation, changes and correction of canonical facts. | Preserved across correction. | Representation and retention; event sourcing is not required. |

### 3.1 Aggregate-boundary alternatives

An account-centered model makes account-local invariants easy to locate, but a transfer or payment touches more than one account. A transaction-centered model naturally groups one economic event and its legs, but must still coordinate account state, concurrent operations and dependent contexts. A hybrid may use separate account and transaction aggregates with an application-level atomic unit of work.

**Open:** Select aggregate roots and their invariants after examining correction, concurrent writes and multi-account operations. Whatever choice is made, an aggregate boundary must not be mistaken for permission to commit half a transfer, payment or financially critical cross-context operation. See Q-04 and Q-05.

### 3.2 Candidate account facts

The conceptual model identifies account ID, name, `ASSET|LIABILITY`, generic type, currency, `ACTIVE|ARCHIVED`, liquidity eligibility, opening state and timestamps. Credit terms may include an informational credit limit and statement/due timing. Their exact representation belongs to the later tactical and commitments design.

Archive retains history and non-zero economic effects. The Available formula includes eligible archived assets; a developer must not silently omit them because they are hidden from ordinary selection. Restore an archived account before ordinary new entry; historical corrections remain possible. Account class cannot change after creation. Eligibility changes still require detailed design. See [CR-LDG-001](../readiness/ledger-product-decisions.md).

### 3.3 Candidate transaction facts

The conceptual model lists `INCOME`, `EXPENSE`, `TRANSFER`, `LIABILITY_PURCHASE`, `LIABILITY_PAYMENT`, `REFUND`, `BALANCE_ADJUSTMENT` and `FEE_INTEREST`, with stable ID, positive transaction amount, effective time, currency, `POSTED|VOIDED` status, affected account legs, optional category and provenance. The engine allows non-negative canonical entered amounts generally; the conceptual transaction rule is stricter (`amount > 0`). Zero opening balances and zero transaction amounts must not be conflated.

Kinds identify economic meaning; direction comes from the kind and account effect, not a negative entered amount. Exact field requirements, permitted account classes, adjustment workflow, and the identity of edit revisions require review. No particular SQL table, API payload or class hierarchy is implied.

## 4. Money, time and state

### 4.1 Money

Canonical money uses integer minor units, never floating point. V1 supports IDR only. Validate the transaction's positive amount and supported currency without inferring how balances or credit limits should be bounded. Rounding for division-derived outputs is governed by the engine; Ledger must not independently invent a rounding policy.

### 4.2 Financial time versus audit time

A date-only transaction belongs to that local financial date in the configured financial timezone. A supplied timestamp is interpreted as its resolved instant in that timezone. Creation/update/audit timestamps serve a different purpose from the economic effective time. Backdating can affect historical budget assignment and downstream calculations; it must not silently rewrite the audit trail.

Financial Calendar resolves cycle membership for consumers requiring it. The frozen Calendar specification's historical floor and result semantics apply; do not assume the optional `ResolveFinancialCycleContext` query exists. An otherwise valid actual transaction may predate the floor; affected cycle-dependent budget results for that date are unavailable. Affected cycle-based historical reports must be marked incomplete or unavailable; do not invent a cycle.

### 4.3 Current financial effect

For an asset account at or after its opening snapshot, the engine defines the snapshot balance plus posted realized legs after that snapshot through evaluation time; voided legs have zero current effect. Earlier transactions remain historical facts but do not change that snapshot or subsequent balance; pre-snapshot balances are not inferred. When a date-only transaction and opening snapshot share a date and their times do not establish order, ask the user to identify before or after; preserve the choice and apply its effect to subsequent balance only when after. Liability outstanding is the engine's floored-at-zero sum of opening liability, recognized purchases and fees, less payments and liability-reducing credits. New liability payments greater than recorded outstanding are rejected with guidance; no separate credit-balance asset is defined.

Balances may be computed or represented using rebuildable derived state, subject to the constitution. Their exact persistence and consistency mechanisms are open. Forecast inputs are assumptions and cannot create posted Ledger facts.

## 5. Transaction semantics

| Economic action | Required actual effect | Must not happen |
|---|---|---|
| Income | Increase receiving asset; newly available delta remains Unallocated by default. | Automatically link Expected Income or assign allocation based on source. |
| Asset-funded expense | Reduce asset; update eligible budget spending and dependent actual state. | Reject merely because truthful reality creates an allocation shortfall or deficit. |
| Owned asset transfer | Apply source and destination effects atomically. | Count the transfer as income, expense or budget spending. |
| Credit/PayLater purchase | Recognize expense at purchase and increase liability. | Decrease an asset merely for the purchase or treat credit limit as cash. |
| Liability payment | Reduce paying asset and liability; settle the relevant obligation as applicable. | Recognize the original purchase expense a second time. |
| Refund/credit | Apply the appropriate asset/liability effect and reverse attributable eligible spending. | Count a refund as duplicate income or reverse more budget spending than attributable. |
| Fee or interest | Recognize expense and liability when contractually incurred under the entered schedule. | Treat repayment as another fee expense or infer an unsupported installment amount. |
| Balance adjustment | Record an explained auditable change to actual account state, show it separately in reports, and start forecasts from corrected actual state. | Automatically classify it as income, expense or category spending. |

This table summarizes intent; source requirements govern detailed formulas. The split between supported operation variants and UI actions remains Q-06.

## 6. Account lifecycle and categories

### 6.1 Account lifecycle

A user can create, edit and archive supported asset and liability accounts. Onboarding may create default Cash and must not require other accounts. Starting Cash is Unallocated until explicitly allocated. Financial eligibility and class are distinct from display or archive status. Archived eligible asset balances remain in Available under `RULE-AVAILABLE-001`.

A change to an account's financial data or eligibility can require recalculation of Available, allocations, STS, forecast and insights. The allowed mutations, their audit effect and treatment of existing transactions remain review items. The opening-balance correction mechanism must not create duplicate actual facts or silently rewrite historical effects.

### 6.2 Category lifecycle

Categories classify relevant expenses and fees and can map to budgets. V1 must not depend on a founder-specific fixed taxonomy. Categories can be renamed or archived while preserving references; history shows the current name and retains rename audit. Archived categories are hidden from ordinary new entry. Merge and deletion remain open. A transaction recategorization must update affected budget spending for its effective cycle, while retaining auditability.

## 7. Candidate commands and outcomes

Names in this catalog describe intent, not approved Kotlin APIs or schemas. Each command must enforce the applicable baseline and participate in the critical transaction boundary. Required data below is conceptual; no optionality beyond the baseline is decided here.

| Candidate command | Relevant input | Successful outcome | Cross-context responsibility |
|---|---|---|---|
| Create/edit/archive account | Account identity where applicable, class/type, financial attributes and user changes | Account state changes; archive preserves effects and history. | Eligibility/opening changes may affect allocation and intelligence. |
| Record income | Receiving asset, amount, effective time, optional explicit Expected Income selection | Posted actual income and updated actual state. | Allocation receives newly available delta; Income Management handles an explicit link. |
| Record expense | Account, positive amount, category, effective time; optional metadata | Actual expense and corresponding account/liability effect. | Planning, Allocation and Intelligence react consistently. |
| Transfer | Source and destination assets, amount, effective time | One economic transfer with both effects committed. | Recalculate eligibility/Available where relevant; never budget it as spending. |
| Record liability payment | Paying asset, liability, amount, effective time and applicable settlement intent | Asset and liability effects plus applicable settlement. | Commitments owns obligation/settlement semantics. |
| Record refund or fee | Original relationship where relevant, account(s), amount, effective time, category where applicable | Actual credit/fee with attributable financial effects. | Budget and commitment impacts depend on the actual relationship and rules. |
| Edit/void/correct transaction | Target identity, requested correction and necessary dependent context | Audit-preserving change with recalculated effects. | Affected contexts must remain consistent, including related settlement/statement. |

Successful, invalid input, domain rejection, conflict/concurrent modification and infrastructure failure must be distinguishable to the caller. Concrete result types and error codes remain design work. A failed financial mutation leaves canonical state equivalent to before the attempt; error messages must not expose sensitive data.

### 7.1 Income and optional explicit claim

Recording actual Income is valid without an Expected Income occurrence. When the user explicitly selects an eligible occurrence, the orchestrated operation records the actual fact and asks Income Management to make the applicable `EXPECTED|MISSED → RECEIVED` transition. Actual amount and time are authoritative. No similar amount, date, description or source may automatically create the association. After unlinking, use EXPECTED if expected time has not passed and MISSED if it has. If intervening changes make the state unsafe to determine, require guided resolution without partial save. Link representation, conflict detection and exceptional lifecycle resolution require Income Management design.

### 7.2 Transfer and payment atomicity

A transfer and a liability payment each affect multiple financial records. Application orchestration and the persistence design must provide the required atomic boundary; success must never be returned for only one effect. Critical downstream state must be consistent on success. An asynchronous event may trigger recoverable secondary work but cannot repair an otherwise incorrect success outcome.

## 8. Corrections and historical changes

A correction may change amount, account, category or effective time, or void an existing transaction. For each affected fact, identify the old and new financial effects and recompute dependent results from authoritative records. The system must preserve audit history and protect dependent relationships; `EC-REF-001` rejects destructive deletion of a record with dependent settlement/reconciliation in favor of the applicable void/cancel/supersede workflow.

| Change | Minimum effects to review |
|---|---|
| Void expense | Remove its current balance and eligible budget contribution; recalculate allocation/shortfall, STS, forecast and insights. |
| Recategorize | Remove old budget contribution and apply new eligible mapping for the relevant financial cycle. |
| Backdate or change date | Re-evaluate affected historical cycle, account state and budget/forecast dependencies. |
| Correct credit purchase across cutoff | Reassign statement/obligation atomically while preserving one purchase expense (`EC-CC-003`). |
| Correct payment with settlement | Reconcile liability and obligation settlement together; never leave an orphan or duplicate settlement. |
| Correct linked actual income | Coordinate actual transaction and expected-income association/lifecycle; apply the approved unlink timing rule or require guided resolution when intervening changes make it unsafe. |

Whether corrections use in-place revision plus audit, reversal and replacement, or another representation is not decided. The implementation is event-driven, not event-sourced. Effective-time changes and physical record edits are distinct from audit timestamps.

## 9. Candidate queries

| Candidate query | Intended result and use | Contract question |
|---|---|---|
| Account details/list | Class, status, financial attributes, eligible balance/outstanding; include archived when requested by authorized consumers. | Filtering, sorting and representation of archived effects. |
| Transaction details/history | Actual posted/voided facts, account effects, financial time and provenance appropriate to use. | Pagination, as-of interpretation and audit access. |
| Actual account position | Asset balances and liability outstanding as of an explicit evaluation time. | Whether temporal queries and consistency guarantees need separate interfaces. |
| Eligible spending facts | Expense, refund and category/effective-time facts needed by Spending Planning. | Filtering, affected-cycle information and correction semantics. |
| Actual financial changes | Authoritative actual information needed by Allocation, Income Management, Commitments and Intelligence. | Purpose-specific contracts and transaction/snapshot consistency. |

Read models may be persisted only under the constitution's rebuild and provenance constraints. They are not a second canonical authority. A stale derived result must not be presented as newly authoritative; failures follow NFR-REL-002.

## 10. Candidate domain events and services

Candidate facts include `AccountCreated`, `AccountChanged`, `AccountArchived`, `TransactionRecorded` and `TransactionCorrected`/`TransactionVoided`. They are illustrative names, not an approved event catalog. Review whether kind-specific facts or a shared fact provide sufficient information without leaking sensitive financial data. Specify identity, occurrence/audit time, versioning, consumers, delivery, failure and recovery in the later catalog. No event is evidence of successful persistence until the encompassing operation commits.

Potential services include account-balance calculation and validation of multi-record transaction effects. Their boundaries, dependency contracts and inputs remain open. The application layer coordinates contexts but must not absorb their business rules. Avoid implementing separate budget, calendar, allocation or STS logic inside Ledger.

## 11. Consistency and persistence constraints

- SQLite and SQLDelight are the locked persistence baseline; domain logic remains behind domain-facing ports and does not depend on SQLDelight internals.
- Canonical multi-record mutations and financially critical invariant enforcement complete synchronously and atomically. Failure must not leave a partial result or orphan a settlement/statement relationship.
- Concurrent writes cannot create duplicate reconciliation/settlement or allocation oversubscription. The exact locking, versioning or serialization mechanism is undecided.
- The model must support rebuilding derived balances and relevant read models from canonical facts and applicable rules, with provenance adequate for audit/debugging.
- In-process events can support secondary reactions. Critical correctness cannot rely on volatile queued work, handler order, an external broker or a transactional outbox.
- Stable identifiers, timestamps and provenance should follow NFR-DATA-005. Audit history is required without prescribing an event store.

## 12. Consolidated invariants for review

| ID | Candidate invariant restating baseline | Source |
|---|---|---|
| INV-LDG-01 | Each economic event is financially recognized once. | DM-PRINCIPLE-001; PRINCIPLE-004 |
| INV-LDG-02 | Actual transactions and forecast assumptions are distinct. | PRINCIPLE-001; RULE-FCST-101 |
| INV-LDG-03 | Canonical money uses integer minor units; a transaction amount is positive. | RULE-MONEY-001–002; DM-TRANSACTION-001 |
| INV-LDG-04 | Owned transfer effects commit together and do not count as spending or income. | FR-TRX-004; EC-TRX-001 |
| INV-LDG-05 | Purchase creates expense and liability; payment does not duplicate the purchase expense. | RULE-CC-101; RULE-CC-103 |
| INV-LDG-06 | Credit limit is never Available; eligible archived assets remain included. | RULE-AVAILABLE-001; FR-ACC-003 |
| INV-LDG-07 | Actual income may be unlinked; only explicit user intent links Expected Income. | FR-INC-003; FR-INC-005 |
| INV-LDG-08 | Actual newly available money remains Unallocated until explicitly assigned. | RULE-ALLOC-101A; FR-TRX-002 |
| INV-LDG-09 | A truthful expense may create a shortfall; it remains recorded and STS is unresolved. | FR-ALLOC-003; EC-TRX-003 |
| INV-LDG-10 | Corrections preserve audit and propagate all applicable consequences. | FR-TRX-005; NFR-DATA-003 |
| INV-LDG-11 | A failed financial mutation leaves no partial canonical change. | NFR-DATA-001; NFR-REL-001 |
| INV-LDG-12 | Cycle resolution belongs to Financial Calendar, not Ledger. | PRD-CYCLE-001; strategic DDD §5.6 |

The IDs above are draft review handles, not an approved decision register. Reconcile them with the source before freeze.

## 13. Worked examples

### 13.1 Transfer

An asset account holds Rp1,000,000 and another holds Rp200,000. Transferring Rp300,000 produces balances Rp700,000 and Rp500,000. The combined balance is unchanged and no category budget is spent. If the destination effect cannot commit, the source effect cannot remain committed. Available may need recalculation when the accounts differ in liquidity eligibility; the example does not assume both are eligible.

### 13.2 Purchase and payment

A credit purchase of Rp800,000 produces Rp800,000 expense and increases liability by Rp800,000, with no immediate asset reduction. A later Rp800,000 payment from an asset reduces that asset and the liability and settles the applicable obligation. It does not create another Rp800,000 purchase expense. Fees, if any, are separately recognized when incurred under the entered terms.

### 13.3 Expected versus actual income

An expected salary of Rp5,000,000 is planning data. Posting actual Income of Rp4,800,000 increases the receiving asset by Rp4,800,000. Without a user-selected link, the occurrence is not marked received just because it resembles the actual Income. With a valid explicit link, Income Management applies the applicable lifecycle transition; no duplicate inflow is created. The newly available amount remains Unallocated until the user assigns it.

### 13.4 Truthful shortfall

Available is Rp1,000,000 with Rp800,000 Flexible and Rp200,000 Reserved. A truthful Rp300,000 asset expense leaves Available at Rp700,000 while existing allocations total Rp1,000,000. The record remains and the Rp300,000 shortfall is surfaced; STS and Daily STS are unresolved until explicit resolution. The system does not silently select an allocation to reduce.

## 14. Review findings and open decisions

These are **open review findings**, not completed audit conclusions. Assign owners and capture decision, rationale, approval date and affected requirements in [the decision register](../readiness/decisions.md).

| Finding | Impact | Next step |
|---|---|---|
| Q-04: Aggregate and persistence design not selected. | Developer cannot safely derive transaction boundaries or correction storage. | Compare candidate roots against multi-account operations, concurrency and audit; document choice and technical ADRs. |
| Q-05: Cross-context correction and atomic transaction contract not specified. | Risk of orphaned settlements, duplicate links or inconsistent critical state. | Model explicit orchestration and failure/rollback for expense, payment, linked income and correction. |
| Q-06: Input variants and validation remain incomplete. | Eligibility, zero states and unlinked refund validation may be guessed. | Apply CR-LDG-001 choices and resolve remaining input rules against the baseline. |
| Pre-floor report implementation needs review. | An allowed backdated transaction lacks a resolvable financial cycle. | Mark affected cycle reports incomplete or unavailable and design precise presentation without inventing a cycle. |
| Refund/adjustment integration needs review. | Later refund links and adjusted actual state affect dependent calculations. | Apply approved reporting and forecast rules; specify atomic coordination and validation. |
| Event/query catalog and reliability contract not selected. | Consumers may depend on volatile events or inconsistent read models. | Define authoritative reads, event payloads, consistency and recovery after context review. |

## 15. Candidate verification matrix

These are planned checks, not test results. Implement them as deterministic tests against approved behavior; extend [the acceptance catalog](../readiness/acceptance-catalog.md) after review.

| Area | Minimum test cases |
|---|---|
| Money and dates | Positive transaction amounts; unsupported fractions/currency; financial timezone; date-only and explicit timestamp; boundary/backdate and historical floor. |
| Accounts | Asset/liability classification; create/edit/archive; archived eligible non-zero asset still contributes to Available; limit excluded. |
| Income | Unlinked income; explicit expected link; late MISSED link; mismatched actual amount/time; no fuzzy auto-link or duplicate inflow. |
| Expense and allocation | Asset expense; budget effect; truthful shortfall retained; explicit resolution; failure leaves no partial write. |
| Transfer | Both legs succeed; injected failure on either side rolls back; no income/expense/budget impact; liquidity eligibility effects. |
| Credit and PayLater | Purchase then payment; fees when incurred; statement cutoff correction; installments use actual schedule and do not duplicate principal/expense. |
| Refund and correction | Attributable refund; recategorization; void/backdate; settlement dependency; no orphan/duplicate; audit remains reconstructible. |
| Consistency | Concurrent conflicting operations; one atomic success or safe rejection; critical dependent views correct on success; secondary reaction failure is recoverable. |
| Rebuild | Recompute balances and affected derived data from canonical facts and compare with saved projections after edits, voids and restart. |
| Privacy and accessibility | Financial data absent from prohibited logs/analytics; relevant error states accessible on both platforms. |

Use existing numerical cases AC-01–07, AC-14 and the frozen engine/edge-case fixtures where applicable. Add missing cases for all reviewed operations rather than treating the examples as exhaustive.

## 16. Freeze criteria

Before promotion from draft: assign domain/product/architecture reviewers; resolve or explicitly defer each finding without leaving a behavior required by the slice undefined; select and document aggregate and operation contracts; complete cross-context consistency review; reconcile any changes to normative product behavior; trace each invariant and test case to an authoritative source; review security, audit, time, failure and recovery; and record the approval and version. Financial Calendar's frozen status does not imply this draft has received an equivalent audit or approval.

## 17. Product decisions accepted — CR-LDG-001

The product owner accepted the recommendations in [CR-LDG-001](../readiness/ledger-product-decisions.md). The associated `FR-LDG-*`, `RULE-LDG-*`, `EC-LDG-*` and `UF-LDG-*` clarifications govern approved product behavior. This tactical document remains a draft: Q-04–06 and concrete cross-context, validation and exceptional lifecycle details still require resolution for affected implementation paths. The technical direction in the change record remains subject to design review.


## 18. Owner-approved product clarification — CR-LDG-002

The owner approved [CR-LDG-002](../readiness/ledger-product-decisions.md). Linked refunds cannot exceed the original purchase; changes invalidating links require explicit resolution without partial mutation. Correcting a wrong account class uses a guided, auditable transaction correction process. Credit-card refunds requiring a positive credit-card balance are unsupported in V1 and must not silently disappear or be recorded as bank income before receipt. Asset opening balances are effective-time snapshots; pre-snapshot activity remains visible but does not apply again to later balances. Snapshot-day date-only ordering uses the user’s explicit before/after choice. A transaction correction crossing the snapshot previews and recalculates its corrected effect; the snapshot amount/date edit is previewed and audited under CR-LDG-004; technical representation remains under review.

## 19. Proposed tactical design for review

This section is a proposed engineering design, not a frozen decision. The developer can review it against Q-04–06 and the constitution without assigning product rules.

### 19.1 Candidate aggregate boundaries

- **Account root:** stable identity, immutable asset/liability class, lifecycle and account attributes, asset opening snapshot and its effective time. Archiving/restoration must preserve history. Balance is derived from the snapshot and eligible posted effects, not a mutable UI field.
- **Transaction root:** stable identity, kind, positive money, effective time, posted/voided state and the complete set of account effects for one economic event. An owned transfer or liability payment contains both effects under a single transaction identity. Audit revisions preserve previous facts; neither physical schema nor a specific history mechanism is mandated.
- **Category root:** stable identity, current name and active/archive state; name changes retain audit; historical references are not erased. Budget mappings remain owned by Spending Planning.
- Refund links and dependency integrity require an explicit consistency boundary that protects the original purchase and all valid linked refunds. Whether represented inside a purchase aggregate or in a transactionally enforced relationship is not yet selected; a read followed by an unprotected write is insufficient under concurrency.

These roots do not authorize independent commits for a financially coupled operation. Application orchestration coordinates the affected roots and supporting contexts within a single local atomic unit of work as required by the constitution.

### 19.2 Operation contract pattern

Each command receives explicit identities, user intent, valid monetary values and financial time. It loads the authoritative state and relevant dependencies, checks domain constraints and conflicting changes, constructs the proposed mutation, coordinates affected contexts, and commits all required canonical changes or none. A result distinguishes invalid input, missing reference, domain/dependency conflict and infrastructure failure; concrete wire types remain deferred. Success means critical state is consistent, not that a background listener has eventually run.

| Operation | Preconditions / dependency checks | Atomic changes and result |
|---|---|---|
| Create or archive account | Valid class, currency, opening snapshot and identity; account status appropriate for operation. | Preserve account history; recalculate applicable Available/allocation/intelligence. Never remove archived eligible balance. |
| Record actual income | Active receiving asset; positive amount; optional explicitly selected expected occurrence must be validated by Income Management. | Commit Ledger fact and any explicit claim; update actual availability and unallocated delta. A failed claim must not partially commit an operation that included that claim. |
| Record expense | Valid account, category and time; do not reject truthful expense solely for negative tracked balance or shortfall. | Commit fact and account effect, refresh eligible budget/critical allocation state; show shortfall/negative status as applicable. |
| Transfer with optional fee | Valid source and destination, positive amounts and explicit optional fee; review whether same-account transfers are rejected. | Commit source/destination effects and selected separate fee atomically; correction previews user-selected effects. |
| Liability payment | Valid paying asset and liability; proposed payment no greater than recorded outstanding; validate associated settlement through Commitments. | Commit payment and valid settlement together; no duplicate expense or over-settlement. |
| Refund or later link | Validate original purchase relationship, currency and total valid linked refunds; require confirmation for later link; check liability-credit representability. | Commit actual refund once; link and recalculate attributable budget only when valid. Reject excess link without dropping or duplicating actual money. |
| Edit/void | Load current transaction and dependencies; preview user-visible impacts; reject or guide resolution if linked refunds/settlements/income make the proposal invalid. | Commit correction, audit and all applicable critical dependent effects together; on any failure preserve prior canonical state. |

Same-account transfer rejection is a candidate validation for review; it is not asserted as approved product behavior. Likewise, the exact handling of a failed optional link, user confirmation after stale previews, and settlement changes must be specified in concrete orchestration contracts before freeze.

### 19.3 Queries and fact notifications

Candidate authoritative reads are account state by explicit evaluation time, transaction and dependency details, auditable history, and eligible actual spending facts for a requested date range. Queries must make unavailable pre-snapshot account balance and unavailable pre-floor cycle information explicit rather than returning fabricated zero values. Consumers that need cycles use the Calendar authority. Concrete pagination and query signatures remain technical choices.

Candidate notifications report account created/changed/archived, transaction posted/corrected/voided, and refund association changed, with stable fact identity and sufficient provenance for consumers. They do not themselves perform required atomic financial coordination. Payload, dispatch, duplicate delivery and recoverability require review before an event catalog is approved; no event sourcing is implied.

### 19.4 Correction and concurrency review

Proposed approach: use stable transaction identity and revision/audit history, validate dependencies from a consistent read, and detect concurrent modifications before commit. A financial unit of work must serialize or reject competing refund links, liability settlements and allocation updates so that invariants cannot be violated by concurrent requests. The underlying SQLite/SQLDelight representation and optimistic versus pessimistic mechanism are deferred. Derived state must be recomputable from canonical facts; stale results must not be represented as current authority.

## 20. Review checklist and remaining freeze blockers

- **Temporal and refund contracts:** implement approved same-day ordering, transaction correction across the snapshot, payment-conflict resolution and full-refund limitation; implement snapshot edit previews and payment-time/later-history validation; implement the CR-LDG-005 user choice for financially significant ambiguous ordering and the unknown-debt guidance; specify ordering-choice invalidation without inventing outcomes.
- **Tactical decisions:** select refund-link consistency boundary, exact operation and error contracts, account eligibility and amount validation, and current-versus-historical outstanding semantics for backdated payments/refunds.
- **Cross-context contracts:** complete allocation, expected-income unlink/conflict, settlement, credit statement and budget correction coordination, with all-or-nothing failure semantics.
- **Audit and verification:** independently review the proposed model against the frozen sources and approved change records, trace every invariant and candidate case, and review restart/rebuild and concurrent-write scenarios. The checklist is not evidence that tests or reviews have happened.
- **Governance:** assign reviewers and record approval/version only after blockers are resolved. This draft is not frozen.

## 21. Owner-approved correction rules — CR-LDG-003

A transaction correction that moves across an asset opening snapshot previews its impact, retains audit/history, recomputes eligible budgets and applies the transaction effect to the post-snapshot balance only if the corrected position is after the snapshot. A proposed backdated credit-purchase correction that would make an existing recorded payment exceed recorded debt is held for explicit user-guided resolution; no existing payment is silently changed and no partial mutation commits. See [CR-LDG-003](../readiness/ledger-product-decisions.md). Snapshot edit preview and payment-time validation are approved under CR-LDG-004; historical starting-information details, ordering-choice invalidation and technical contracts remain review items. This clarification does not freeze the draft.

## 22. Technical audit — initial pass

**Status:** Initial review, not an independent approval or freeze audit. This is a documentation review, not execution of application tests. Findings describe evidence and impact; an item is not resolved simply because it has a suggested approach. Review date: 2026-11-14. Reviewer: documentation author; independent reviewer unassigned.

| Finding | Severity | Evidence and impact | Next action |
|---|---|---|---|
| AUD-LDG-001 — Refund link concurrency | Blocking | Proposed transaction roots and refund associations have no selected shared consistency boundary. Two concurrent links could individually fit but together exceed the purchase. | Select an application-unit-of-work invariant and conflict/serialization contract; test concurrent attempts and rollback. Q-04/Q-05. |
| AUD-LDG-002 — Liability history and snapshot correction | Blocking | `RULE-BAL-002` floors a calculated outstanding while payment/refund restrictions and CR-LDG-003 depend on the relevant debt. Historical evaluation, event ordering, and effects of editing opening state are not fully specified. The floor must not erase evidence of an invalid dependent payment or credit. | Define authoritative as-of inputs and validation scope; reconcile any user-visible ambiguity before approval. Q-05/Q-06. |
| AUD-LDG-003 — Critical cross-context operations | Blocking | Draft lists affected contexts but does not yet provide complete preconditions, failure/rollback and resulting state for settlement, income unlink, refund relink, statement correction and budget updates. | Complete purpose-specific orchestration contracts and test successful commit, conflict, stale preview, and rollback. Q-05. |
| AUD-LDG-004 — Snapshot ordering and corrections | Blocking | `RULE-BAL-001` now requires explicit same-day user ordering; correction of the snapshot itself, ordering explicit timestamps at the exact boundary and choice invalidation on edit are not defined. Pre-snapshot balance cannot be fabricated. | Define precise inputs, preview, preservation/reconfirmation and as-of results, with product reconciliation where outcomes need a choice. |
| AUD-LDG-005 — Candidate command validation | Blocking | Same-account transfer, account eligibility changes, some category/adjustment inputs and refund association constraints lack fully reviewed contracts. A transfer table previously assumed distinct accounts although the text called it undecided; that inconsistency was corrected in this review. | Map each rule to a normative source or seek a product decision; define result semantics without inventing behaviors. Q-06. |
| AUD-LDG-006 — Actual versus expected income wording | Reconciliation review | `RULE-STATE-001` says expected income cannot increase balance before an explicit link; `FR-TRX-002` and `RULE-FCST-103A` say posted actual income increases actual state even if unlinked. The correct interpretation appears to be that the *expected assumption* adds nothing while actual posted income is valid independently; the wording could be misread as blocking unlinked actual income. | Review and reconcile wording across affected frozen sources without changing the approved optional-link behavior. |
| AUD-LDG-007 — Unsupported positive card credit | Product limitation / review | CR-LDG-002 and `FR-LDG-011` require surfacing a card refund that V1 cannot represent; recording the later bank return only when actually received does not record the intervening card credit. | Ensure UI never claims the ledger is complete during this period; review linkage and impact when a bank refund later arrives. Do not invent a credit asset. |
| AUD-LDG-008 — Event and query contracts | Blocking for freeze | Names, payloads, delivery/failure behavior and consistent reads remain candidates. Secondary events cannot be the sole mechanism for canonical correctness. | Publish purpose-specific contract decisions, clearly identifying essential reads and noncritical notifications; preserve constitution's no event sourcing/outbox constraints. |
| AUD-LDG-009 — Frozen baseline reconciliation | Review required | CR-LDG-001–003 introduced new financial behavior and revised asset balance semantics. The current FR matrix is explicitly a first-pass family mapping, not a reviewed per-clause audit. | Check every affected normative document, formula, flow and scenario; update change-control records and traceability before freeze. |

### 22.1 Suggested audit scenarios

1. Two refunds compete for the last Rp200,000 refundable amount; at most one valid set commits, and a failed link does not create a second refund inflow.
2. A transaction correction crosses the opening snapshot; compare before/after balances, budget effects, user-visible preview and audit, including a date-only same-day explicit choice.
3. A proposed backdated purchase correction invalidates an existing payment; preserve the old purchase, payment, bank balance and settlement until a valid set of changes commits atomically.
4. Explicitly linked salary is voided before and after expected time; verify EXPECTED/MISSED, no partial writes on unsafe intervening lifecycle change and no duplicate forecast event.
5. Archive an eligible nonzero account, rebuild Available and allocation state after restart, and verify it remains included while new entry requires restoration.
6. Transfer with a charged fee: simulate failure at each financial mutation, then confirm selected void/refund behavior and no partial update.

These scenarios are proposed checks, not completed tests. Explicit snapshot-date and historical liability edge cases must be completed before they can be treated as executable test oracles.

### 22.2 Audit disposition

The draft is **not ready to freeze**. Some SQL schema, indexing, implementation-class names and physical event mechanisms may be deferred to later design, but unresolved domain invariants, user-visible behavior, critical cross-context consistency and a coherent query contract cannot be hidden behind that deferral. Product decisions accepted in CR-LDG-001–003 remain accepted; this audit does not reverse them.

## 23. Owner-approved temporal decisions — CR-LDG-004

Asset opening snapshot edits require an impact preview, transaction relationship review, confirmation, retained audit and recalculation; ask the user for before/after when same-day ordering is unclear. Backdated credit-card payments are checked against recorded debt at payment time and throughout affected later history; an unsupported positive card-credit conflict requires correction without partial save. Indeterminate debt or ordering is surfaced rather than guessed. See [CR-LDG-004](../readiness/ledger-product-decisions.md). The owner subsequently approved the relevant same-day date-only ordering choice and unknown-debt guidance in CR-LDG-005. AUD-LDG-002 and AUD-LDG-004 are not closed by product approval alone.

## 24. Owner-approved historical credit-card decisions — CR-LDG-005

If the order of same-day date-only credit-card transactions affects whether a payment is valid, obtain and preserve the user's choice; do not ask when the order cannot affect the financial result. When debt immediately before an old payment is unknown, show that uncertainty and guide the user to provide or correct the needed starting information before validating/saving the payment. Do not infer that sufficient debt existed. See [CR-LDG-005](../readiness/ledger-product-decisions.md).

Candidate operation contract: resolve known historical facts and relevant user-provided order, derive debt at the payment time and check affected later history. If debt cannot be established, return an explicit history-unavailable outcome with no canonical mutation; if a required order is missing, return a needs-ordering outcome with no canonical mutation. After user supplies information, re-read/revalidate and commit only if all critical invariants hold. Exact result names are proposals, and the starting-information flow and conflict/revalidation mechanics still require review. The owner subsequently selected equal-explicit-timestamp handling in CR-LDG-008; liability opening snapshot mechanics still require review.

## 25. Owner-approved historical balance and order revalidation — CR-LDG-006

The user may provide an earlier known card-debt amount and effective date through a guided flow. The flow identifies later transactions to check, previews impact and preserves audit, without counting debt twice. The owner subsequently approved a historical checkpoint comparison with the existing opening balance, without adding the balances, and discrepancy/indeterminate outcomes in CR-LDG-007. This does not extend the asset snapshot formula to liabilities. Technical interval, persistence and concurrency review remain. See [CR-LDG-006 and CR-LDG-007](../readiness/ledger-product-decisions.md).

When an edit affects card history, revalidate it and retain a saved ordering choice only if still applicable. If determining payment validity needs a new order, request it before saving; resolve conflicts without partial mutation. A candidate technical contract records the identities and relevant temporal facts the user's choice relates to, checks them against the current authoritative revision during revalidation, and treats stale or inapplicable choices as requiring renewed input. This is proposed for review, not a frozen persistence schema. Re-read and validate the full affected history in the same critical consistency boundary before commit.

AUD-LDG-002 remains open: the checkpoint comparison interval, same-instant ordering, affected-history calculation and conflict contracts require technical review. AUD-LDG-003 and AUD-LDG-008 remain open for detailed orchestration and query contracts. Product acceptance does not close the audit or freeze the document.

## 26. Owner-approved historical checkpoint — CR-LDG-007

Use an earlier known card-debt balance as a historical checkpoint, not an additional debt. Compare determinable intervening effects with the existing later opening balance, which remains the starting point at and after that boundary. If a determinable comparison differs, show the discrepancy and mark affected historical calculations incomplete or unavailable until resolved; do not silently alter balances or create a transaction. If intervening facts or ordering are insufficient, report indeterminate instead of fabricating a numerical discrepancy. An old payment still requires established debt at its financial time. See [CR-LDG-007](../readiness/ledger-product-decisions.md) and `RULE-LDG-005`.

Technical review must define interval inclusion at boundaries, stale preview/concurrent edits, effective-time ordering, validation against intervening payments, and checkpoint/audit representation. AUD-LDG-002 remains open pending those contracts and review. No executable tests have been run.

## 27. Consistency contracts — technical review proposal

**Status:** Proposed technical design; not frozen or a claim of independent review. These contracts implement existing decisions and do not resolve unspecified user-visible outcomes. `Unknown` below is an explicit domain outcome, never an amount of zero. Concrete Kotlin signatures, storage schema and concurrency technique remain later design details.

### 27.1 Authoritative facts and calculated results

- Ledger is authoritative for account and actual transaction facts, opening facts, checkpoint facts and the user's relevant ordering decisions. It must be possible to identify the revision of each fact used in a result. Other contexts retain their own authoritative facts and invariants.
- Account balances and historical comparisons are derived, not directly editable. A result identifies its effective evaluation time, inputs, applicable ordering and status. Missing or conflicting inputs return an explicit unavailable, indeterminate or conflict outcome; they do not produce an authoritative number.
- At or after the later opening, use that opening and applicable subsequent effects; an earlier checkpoint is never added to that opening. Before the opening, a historical calculation requires an applicable earlier checkpoint and sufficient facts. Do not infer pre-snapshot asset balances.
- To compare an earlier debt checkpoint with a later opening, start from the earlier checkpoint and apply only the effects that belong between the two boundaries. The later opening is the comparison target, not another inflow. When ordering at either boundary affects the result, apply the user’s explicit before/after choice under CR-LDG-008; if a required order cannot be established, return an indeterminate comparison.
- A determinable difference returns both recorded and calculated values and the difference; mark the affected historical calculation incomplete/unavailable until resolved. If missing data prevents a trustworthy calculation, return indeterminate rather than a numerical difference. Neither outcome creates a transaction or changes the recorded balances.

### 27.2 Preview, confirmation and commit

A preview is a read-only calculation over an identified consistent set of inputs. It reports the proposed changes, affected account effects, relationships, ordering questions and known downstream impacts. An unavailable result, existing discrepancy or unsupported credit condition must not be presented as a verified result. A preview is not a committed transaction or a promise that the facts will remain unchanged.

On confirmation, the application starts an atomic unit of work, rereads the authoritative inputs and verifies that the preview's relevant facts and ordering decisions are still applicable. If anything relevant has changed, it returns a stale/conflict outcome requiring a new preview. It does not silently reuse an obsolete approval. It then asks each owning context to enforce its rules, applies the complete financially critical mutation and audit changes, and commits together. Any failed validation, conflict or persistence failure leaves the pre-operation canonical state intact.

A database transaction or equivalent port must serialize or detect conflicting writes affecting the same invariant. The design must not rely on a prior UI validation, unprotected read-then-write, background handler, wall-clock timing or assumed single-user sequential access. Whether optimistic versions or another suitable SQLite/SQLDelight mechanism is selected is deferred; the guarantee is required.

### 27.3 Refund and settlement conflict boundaries

For a linked refund, the unit of work checks the original purchase, all currently valid linked refunds and the proposed change together. It commits the link and affected budget result only if the total remains within the purchase amount. Two concurrent attempts that compete for the remaining amount cannot both commit if their total would exceed it. Failed linking of an already-posted unlinked refund leaves that actual refund intact; it does not create another inflow or delete the existing fact. Creating a new refund and linking it as one explicitly requested operation either commits the valid operation as a whole or leaves no partial mutation.

A payment's Ledger fact, account effects and applicable Commitment settlement are coordinated together. A correction checks the existing settlement and affected liability history. Reject or require guided resolution when the confirmed operation would create an unsupported credit, invalidate a dependent payment or exceed a valid settlement; do not silently alter a payment. The Commitments context must own its settlement validation. Detailed supported settlement variants and exceptional lifecycle resolutions remain open until that context's contract is reviewed.

### 27.4 Checkpoint and order revalidation

The checkpoint preview identifies the earlier recorded amount and effective time, the later opening and the intervening posted facts and relevant user ordering choices. It supplies `Matched`, `Discrepancy`, or `Indeterminate` as conceptual outcomes; exact API names are not decided. A `Discrepancy` may show an amount only when all required inputs are sufficient. The later opening still controls the balance at and after its boundary. A discrepancy is not silently resolved by changing the opening or synthesizing a fee, expense, payment or adjustment.

Ordering choices must be tied to the facts and temporal circumstances they describe. When an affected transaction or boundary changes, the operation revalidates the choice. If it no longer applies and the order matters to payment validity, request user input before committing; do not reuse it by default. Preview and confirmation use the same stale-input protection. The precise rule for deciding whether a previous choice still applies is a review item; no ordering is inferred from record IDs, database insertion order or event-handler order.

### 27.5 Reads, notifications and recovery

| Interface purpose | Required observable contract |
|---|---|
| Actual account position | State the evaluation time and relevant source revision; return a valid value or an explicit unavailable/conflict state. Archived eligible assets are included when computing Available. |
| Historical checkpoint comparison | Distinguish match, determinable discrepancy and indeterminate inputs. A consumer must not treat an incomplete comparison as a reconciled result. |
| Actual transaction and dependencies | Return the facts and relationships needed for correction, audit and validation; relevant contexts remain authoritative for their own state. |
| Eligible spending facts | Supply realized, correctly categorized facts and corrections needed by Spending Planning; use Financial Calendar for cycle resolution rather than implementing the calendar in Ledger. |
| Secondary notification | Signal a committed fact for noncritical reactions; failure must not change committed truth, and secondary state must be identifiable and recoverable. Notifications are not the only mechanism for financially critical consistency. |

A reproducible rebuild must yield equivalent financially critical results from canonical facts, applicable settings and explicit time/order choices. Derived-cache failure must not silently replace authoritative facts. Event delivery details and concrete event payloads remain review items; neither an event store nor an outbox is authorized by this proposal.

### 27.6 Review cases and disposition

| Candidate verification | Expected result |
|---|---|
| Two concurrent refund links each fit individually but exceed the purchase together | At most a valid combination commits; no duplicate inflow, silent link or lost refund. |
| Preview correction, then modify a dependency before confirmation | The stale preview is rejected and refreshed; no partial correction commits. |
| Rp700,000 earlier checkpoint plus complete intervening effects yields Rp950,000 against Rp1,000,000 opening | Show Rp50,000 discrepancy and incomplete/unavailable affected history; preserve both balances and create no transaction. |
| Same comparison with unknown intervening time/order | Show indeterminate, not a fabricated Rp50,000 discrepancy. |
| Earlier ordering choice becomes inapplicable after an edit | Ask again only if ordering now affects validity; preserve existing canonical facts if user cancels or validation fails. |
| Rebuild after a checkpoint correction or archived-account change | The rebuild agrees with the correctly committed canonical facts and visibly preserves unavailable or incomplete states. |

These are candidate tests, not test results. AUD-LDG-001–004 and AUD-LDG-008 still require contract review, boundary completion and verification. Same-instant checkpoint ordering, liability opening-state completeness, unsupported card-credit representation, exact applicability of ordering choices and baseline reconciliation remain open. Do not freeze this document on the strength of this proposal alone.


## 28. Equal-time decision and baseline reconciliation — CR-LDG-008

The owner selected [CR-LDG-008](../readiness/ledger-product-decisions.md): when equally timestamped financial facts, or a transaction and balance boundary, have an order-dependent result, request and preserve the user's real ordering. Revalidate it after affected changes. If the required order cannot be established, surface indeterminate and do not commit an operation requiring a definite answer. Do not request an arbitrary order where the result is order-independent. `FR-LDG-017`, `RULE-LDG-006` and `EC-LDG-028`–`030` capture the behavior; the existing date-only rule remains intact. Do not mutate timestamps to represent a user ordering.

AUD-LDG-006: `RULE-STATE-001` has been clarified against the authoritative actual-income requirement: actual posted income updates actual state whether or not an expected occurrence is linked; expected income itself never creates actual money. This is a wording reconciliation, not an authorization to auto-link. Reconciliation still requires audit review of related sources and scenarios.

The technical model must use the persisted user relationship when ordering affects checkpoint inclusion, validate that choices describe the current facts, and return indeterminate for an unresolved relevant tie. Physical storage and exact dependency/version representation are technical design details. AUD-LDG-002, 004 and 008 remain open for boundary and consistency review; this change does not imply independent approval, executed tests or frozen status.

## 29. Consolidated tactical model — review candidate

**Status:** FROZEN v1.0 by owner instruction, 2026-11-18, subject to open findings in §30. This section consolidates and supersedes provisional alternatives in §§3, 7, 9, 10, 19 and 27 where they overlap. Product and financial baseline remains authoritative. Audit findings in §22 record the state at the earlier review date; §30 records the current disposition. No SQL schema, endpoint, exact dispatch implementation, platform-specific framework API, completed tests or independent approval is implied.

### 29.1 Responsibility and aggregate boundaries

The candidate uses three roots: **Account**, **Transaction**, and **Category**. Account owns class, lifecycle, eligibility, opening balance fact and account-specific terms. Transaction owns its kind, identity, status, economic time, amount and the complete set of account effects belonging to one economic event; transfer is one transaction with source and destination effects. Category owns its name and archive state, with historical references preserved. Audit revisions are durable records associated with a mutation, not an event-sourced system of record. Financial balances are derived from opening facts and valid posted effects.

The candidate selects an **application-level atomic consistency boundary**, not a single giant aggregate, for multi-root changes and financial dependencies. A transaction-local unit of work includes the authoritative affected Ledger facts, any required user ordering/link intent, and the owning contexts' critical canonical changes. Within that same commit it protects all refund links to the original purchase, enforces the maximum refundable amount and validates relevant payment/settlement constraints. Concurrent commands must serialize or detect a revision conflict and retry/re-preview; two valid individual reads may not both commit an invalid combined total. This is an invariant and integration contract, not a choice between optimistic locking and database serialization.

### 29.2 Temporal and checkpoint evaluation

Each calculation takes an explicit evaluation time and the relevant canonical effective financial times, opening/checkpoint facts and applicable user ordering choices. For asset balances at/after the opening, include the opening once and only effects established as after it and no later than evaluation. For card balances at/after the later opening, include the later opening once and applicable later effects; an earlier card checkpoint is not added. For a card checkpoint comparison, apply effects established as after the earlier checkpoint and before the later opening; an event tied to either boundary belongs to the relevant side according to the approved user ordering when it affects the result. Void effects contribute zero. These are logical boundaries, not invented sub-second timestamps.

A comparison is `MATCHED` if complete inputs agree, `DISCREPANCY` with the determinable difference if complete inputs disagree, or `INDETERMINATE` if the necessary history or material ordering cannot be established. Distinguish an unknown as-of account balance from a known zero. A discrepancy never creates a transaction or changes either recorded balance. An older payment cannot be committed as valid without established applicable historical debt; if unsupported credit would arise at the payment or in affected later history, require explicit resolution without partial mutation.

For equal explicit timestamps or financially material same-day date-only ordering, retain an explicit user relationship to the relevant facts rather than synthesizing timestamps. A choice is reusable only while the same identified facts, effective-time relationship and meaning of the choice remain unchanged; any change that can alter those circumstances invalidates it and triggers revalidation. If no ordering can change the financial result, no user choice is required solely for ordering. If the user cannot determine a required ordering, return indeterminate and do not commit a use case requiring a definite outcome. Validate the logical ordering consistently; contradictory choices cannot become authoritative.

### 29.3 Required mutation outcomes

| Use case | Required validation and authoritative effects | Non-success outcome |
|---|---|---|
| Account create/edit/archive/restore | Validate account class, supported money and dates; preserve class identity and archive history. Asset opening edits require preview, confirmation, audit and dependent recalculation. Archived eligible balances still contribute to Available. | Invalid input, unavailable history, stale preview or conflict leaves previously committed state unchanged. |
| Income with optional explicit expected occurrence | Record actual income once. If user selected an occurrence, coordinate Income Management's claim in the same critical operation. Newly available delta remains Unallocated. Unlinked actual income remains valid. | An explicitly requested claim that fails cannot leave half of the requested combined operation committed; user may submit a separate unlinked income action. |
| Expense and adjustment | Validate entered values and account use, post actual effect, and recalculate relevant budget and allocation status. An explained adjustment remains distinct from ordinary income/expense. Record truthful negative asset balance and show shortfall if applicable. | Invalid commands produce no mutation; financial shortfall resulting from truthful reality is an accepted surfaced outcome, not a reason to erase the event. |
| Transfer and optional fee | Require different source and destination. Commit both transfer effects and any selected fee expense together; fee is distinct from transfer principal. | Same-account or other invalid request rejects without partial effects. Correction previews user-selected transfer/fee results. |
| Payment and settlement | Validate debt at effective time and affected later history, necessary order and an applicable settlement with the Commitment owner. No unrepresented card credit or duplicate purchase expense. | Unknown debt/order, unsupported credit, stale approval or settlement conflict returns an explicit outcome and commits nothing for the requested operation. |
| Refund or association | Retain an unlinked actual refund as its own fact; a confirmed link updates eligible budget spending without creating an additional inflow. Check all valid links together against original purchase and supported liability representation. | Failed link of an existing refund preserves that existing fact; a newly requested create-and-link operation commits both or neither. |
| Correction or void | Read dependencies, preview, obtain required user choice, revalidate versions/order and apply all affected canonical facts and critical calculations together. Protect linked refunds, income claims, statements and settlements. | Invalidated ordering, dependency conflict or unknown time requires user resolution; preserve pre-operation state on cancellation/failure. |
| Historical card checkpoint | Preview checkpoint, comparison and affected transactions. Keep the later opening unchanged unless the user separately confirms an approved correction. Return MATCHED, DISCREPANCY, or INDETERMINATE; mismatched/unknown historical calculations are not represented as reconciled. | If payment validation requires unavailable historical debt, do not validate/save that payment. Creating the checkpoint itself must not invent a transaction or partial linked mutation. |

### 29.4 Query and event integration contracts

Authoritative query families are account state with evaluation time and explicit available/unavailable status, transaction with current status and dependency references, eligible actual spending facts for a requested date range, historical checkpoint comparison with matched/discrepancy/indeterminate status, and auditable change history. Each consumer obtains cycle information from Financial Calendar rather than recalculating its boundaries. Query results involving more than one authoritative context must use a coherent committed view; inability to obtain it cannot be represented as a fresh authoritative amount.

Notifications report completed committed facts, including account and transaction mutations and association changes. They are in-process and may support noncritical reactions; they cannot substitute for any required critical transaction work. They must carry sufficient stable identity and provenance to identify the fact and recover secondary state, without including sensitive data in logs/analytics. Delivery, duplicate handling and handler-failure strategy belong in the later event catalog; a handler failure must not rewrite a committed fact. No event-sourcing, external broker or transactional outbox is introduced.

### 29.5 Failure, audit and rebuild

Commands distinguish invalid input, domain conflict, missing or indeterminate history/order, unsupported positive card credit, stale preview/concurrent change and infrastructure failure. These are conceptual result categories, not public API names. A financially unfavorable but truthful expense is accepted and surfaced. Audit retains identity, relevant before/after facts, user ordering and link intent and originating operation without relying on replay as the current source of truth. Derived data can be rebuilt deterministically from canonical state and explicit effective time/order. Restore and migration must preserve the same invariants and must not claim unreconciled history is complete.

## 30. Review disposition and handoff

**Status:** The owner froze the consolidated baseline v1.0 on 2026-11-18 with the findings below still open. This is not independent approval, not evidence of implemented behavior, and not a release or implementation-readiness sign-off. This disposition supersedes the earlier dated audit table's wording about current freeze status while preserving that table as audit history.

| Finding | Current disposition | Remaining verification before freeze |
|---|---|---|
| AUD-LDG-001 | A cross-root atomic refund consistency contract is selected in §29.1 and §29.3. | Review and implement a concurrency test demonstrating that invalid combined links cannot commit. |
| AUD-LDG-002 | Owner-selected checkpoint, historical-payment and equal-time rules are mapped in §§28–29. | Review liability opening provenance/completeness and checkpoint interval edge cases against all baseline formulas; verify no floor conceals an invalid payment. |
| AUD-LDG-003 | §29.3 gives per-use-case all-or-nothing boundaries. | Reconcile concrete expected-income, commitment, statement and budget contracts when their tactical designs are written; verify coordinated rollback. |
| AUD-LDG-004 | Owner-selected boundary ordering and snapshot correction behavior is reflected in §29.2. | Verify exact before/after as-of query semantics and stale ordering-choice invalidation in review cases. |
| AUD-LDG-005 | Same-account transfer has a delegated recommended decision in CR-LDG-009; existing account/category rules are linked. | Review other operation-specific input rules and any remaining behavior not fixed by baseline; do not assume unlisted provider-specific behavior. |
| AUD-LDG-006 | Financial-engine wording now explicitly preserves unlinked actual income. | Review cross-document wording and an unlinked-income regression scenario. |
| AUD-LDG-007 | Unsupported card credit remains an explicit V1 limitation, not a silently represented balance. | Review incomplete-state communication and later actual bank-refund association; do not invent card credit. |
| AUD-LDG-008 | §29.4 specifies required read purposes and notification boundaries. | Complete concrete event/query catalogs during cross-context design; verify failure and rebuild semantics. |
| AUD-LDG-009 | Owner decisions and associated requirement, scenario and flow IDs are recorded. | Complete clause-level baseline and traceability audit; group-level mappings are not evidence of full coverage. |

Required decision scope for this candidate is the Ledger domain and contracts; implementation schemas, package names, exact transport types, synchronization and unrelated domain details may remain deferred within the architecture constitution. In contrast, explicitly unresolved user-visible or financially critical behavior cannot be silently deferred into implementation. The owner requested freezing with these findings open. Before implementing an affected path, complete its remaining review, reconcile actual source conflicts through change control, and record the responsible review and disposition. Proposed tests and author review must not be described as an independent audit or a passing test suite.

## 31. Requirements reconciliation and review guide

This is an author-performed, scoped review of the Ledger-specific clarifications, not a certified clause-level audit of every V1 requirement. It gives the product owner a navigable review path and records what must still be verified.

| Product decision or rule | Primary requirements / rule | Review evidence and remaining detail |
|---|---|---|
| Actual income and explicit expected-income link | `FR-TRX-002`, `FR-INC-003`, `RULE-STATE-001`, `RULE-FCST-103A` | AC-03 and AC-14; wording reconciled so actual income is valid unlinked. Concrete unlink exceptions and Income Management contracts remain to review. |
| Archive, account class and category history | `FR-LDG-001`, `FR-LDG-006`, `FR-LDG-010`, `RULE-AVAILABLE-001` | AC-19 and EC-LDG-001/007. Guided wrong-class correction requires detailed end-to-end scenarios. |
| Balance adjustments and truthful negative balance | `FR-LDG-002`, `RULE-LDG-001`, `RULE-LDG-003` | AC-20 and AC-25; verify allocation outcomes and recovery across account eligibility cases. |
| Transfer, fee and correction | `FR-TRX-004`, `FR-LDG-008`, `FR-LDG-018` | AC-02, AC-22 and AC-44; verify every failure point, linked fee choice and same-account rejection. |
| Refund attribution, limits and unsupported card credit | `FR-LDG-005`, `FR-LDG-009`, `FR-LDG-011`, `RULE-LDG-003` | AC-24, AC-27 and AC-30; concurrent link and later actual bank-refund association require further reviewed scenarios. |
| Asset opening and historical transaction | `FR-LDG-007`, `FR-LDG-012`, `RULE-BAL-001`, `RULE-LDG-004` | AC-23, AC-28, AC-29, AC-31 and AC-33; check snapshot/cycle interaction and historical as-of uncertainty. |
| Liability payment, corrections and older debt | `FR-LDG-004`, `FR-LDG-013`–`016`, `RULE-BAL-002`, `RULE-LDG-002`, `RULE-LDG-005` | AC-21 and AC-32–40; inspect all opening/checkpoint boundary cases and reconcile Commitment-owned settlements. |
| Ordering and equally timed facts | `FR-LDG-017`, `RULE-LDG-006` | AC-41–43; review multiple-way ordering, contradictory choices and stale revalidation before treating as test-complete. |
| Audit and atomic operations | `FR-LDG-003`, `NFR-DATA-001`–`005`, `NFR-REL-001`, constitution §§12–14 and 17 | §29 defines candidate contract; executable rollback, rebuild, concurrency and restart tests do not yet exist in this specification repository. |

Review CR-LDG-001–009 against the product requirements, engine, conceptual domain model, flows, analytics privacy and architecture baseline. Mark any contradiction with exact references and apply change control; do not assume the first-pass traceability matrix proves clause-level completeness. NFRs applying to server-backed or authenticated deployments are conditional and do not add those services to the local-first V1 architecture. Security, encryption at rest and user-controlled backup/restore remain subject to their applicable locked requirements.

## 32. Owner freeze record

On 2026-11-18 the product owner requested that the current Ledger documents be frozen. This is an explicit approval of the current §29 tactical baseline and the previously selected product decisions, including CR-LDG-008 and CR-LDG-009. The owner is recorded by role because no personal identity was provided. This record does not claim the owner conducted an independent technical audit.

The open findings in §30, Q-04–06, the broader clause-level reconciliation and executable verification remain outstanding. In particular, do not infer unsupported card-credit handling, outstanding cross-context settlement/link behavior, or incomplete temporal results. Any operation whose behavior cannot be determined from the frozen requirements and approved contracts remains blocked from implementation until the missing rule or contract is explicitly resolved. Existing specifications, governance and release gates remain authoritative. Subsequent changes require the documented change-control process.
