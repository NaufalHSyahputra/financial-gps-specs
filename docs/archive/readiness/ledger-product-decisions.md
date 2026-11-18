# Ledger product decisions — change record CR-LDG-001

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. **Scope:** V1 Ledger product behavior below. **Technical design status:** Draft; acceptance does not freeze the tactical model or approve an ADR. This record does not claim independent review or completed implementation.

The functional requirements, engine, edge cases and user flows contain the corresponding clarifications under `FR-LDG-*`, `RULE-LDG-*`, `EC-LDG-*` and `UF-LDG-*`. Where a behavior was already frozen, this record clarifies it. Where it was previously unspecified, it records an owner-approved requirements change. No source changes Financial Calendar's historical floor. Requirements not affected by this record remain in force.

| ID | Owner-approved direction |
|---|---|
| D-LDG-01 | Archive hides the account from ordinary new transaction selection; restore before a new transaction. Preserve history and non-zero balance, including eligible Available contribution; historical corrections remain possible. |
| D-LDG-02 | Account class cannot be changed after creation. Create a correct account and use a guided correction process for affected history; do not reinterpret facts silently. |
| D-LDG-03 | Support an explained, auditable balance adjustment, displayed separately from ordinary income and expense and not automatically categorized as either. Recalculate actual state and allocation consistency. Show adjustments separately in reports and use the corrected actual balance as the forecast starting point. |
| D-LDG-04 | Allow edit and void with audit history and dependent recalculation; guide the user through affected dependencies. Existing restrictions on destructive deletion remain. |
| D-LDG-05 | Permit truthful asset-funded expenses that result in a negative tracked balance; surface it and suggest review. Eligible account contribution to Available remains floored at zero; apply existing allocation shortfall rules. This is not an overdraft or credit facility. |
| D-LDG-06 | Reject a new liability payment greater than recorded liability outstanding; explain the limitation and suggest checking for missing or incorrect transactions. A separate credit-balance representation is not supported by this decision. |
| D-LDG-07 | Default to a refund explicitly linked to its purchase and reverse attributable eligible spending. Allow a clearly labeled unlinked refund as a separate report item, neither ordinary income nor expense, without an automatic budget change. Later linking requires confirmation and recalculates the affected budget with audit history. |
| D-LDG-08 | Support category rename and archive without deleting historical references. Hide archived categories from ordinary new entry; display the current name in history and retain an audit of renames. Category merge and destructive deletion remain unspecified. |
| D-LDG-09 | Permit actual transactions before Calendar's historical floor, subject to valid transaction rules. Do not invent a cycle; mark affected cycle-dependent budget results unavailable for the date. Show the transaction and account effect; affected cycle-based historical reports are explicitly incomplete or unavailable, without inventing a cycle. |
| D-LDG-10 | Support a transfer with an optional separately recognized fee expense entered in the same flow. Transfer remains a transfer; fee is spending. Show linked transfer and fee together, preview edits and voids, and let the user choose each resulting change; commit the chosen changes together. On void, suggest voiding both while permitting the fee to remain if actually charged. |
| D-LDG-11 | When a linked actual income is edited or voided, show the linked occurrence and ask the user to confirm the appropriate correction; update dependent records consistently and atomically. After an unlink or void removes the supporting link, derive EXPECTED if the expected time has not passed and MISSED if it has; if intervening changes make the state unsafe to determine, require guided resolution and do not partially save. |
| D-LDG-12 | Recommend separate account and transaction concepts, atomic financial mutations, auditable corrections, conflict detection, rebuildable balances and purpose-specific reads. These are technical design direction, not approved aggregate boundaries or implementation contracts. |

## Remaining follow-up and review

- Q-04–06 remain open for aggregate design, cross-context atomicity and unchosen validation details. In particular, resolve actual-input rules for refunds and adjustments without inventing classification rules.
- Owner approved follow-up choices on 2026-11-14: refund reporting and later linking, adjustment reporting and forecast starting state, pre-floor report completeness, transfer-fee correction choices, and ordinary income unlink timing. Resolve concrete integration, validation and exceptional lifecycle contracts during tactical review; do not infer unsupported cancellation transitions.
- Update precise traceability and golden fixtures for these new identifiers, review the linked documents and obtain domain/architecture review. The product owner approved direction; no other approval is implied.

## Additional product decisions — CR-LDG-002

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Technical implementation and review remain outstanding.

| ID | Owner-approved direction |
|---|---|
| D-LDG-13 | Reject a refund link when the total valid linked refunds would exceed the original purchase. Explain the remaining linkable amount; never silently classify any excess. |
| D-LDG-14 | If correcting or voiding a purchase would make its linked refunds invalid, show the conflict and require explicit resolution before committing any affected change; do not silently void, reduce or unlink refunds. |
| D-LDG-15 | Correct an account created with the wrong class by creating the correct account and guiding explicit corrections of affected transactions, previewing effects and preserving audit/history; archive the old account when appropriate. Do not silently reinterpret all facts. |
| D-LDG-16 | V1 cannot represent a positive credit-card balance. If a refund to a fully paid card would require one, explain the limitation and do not silently discard money or record an unreceived bank refund. Guide the user to record the actual bank refund when it is returned, or correct erroneous history. If a card refund exceeds outstanding, do not record only the covered portion or discard the remainder; show that the full refund cannot be represented. |
| D-LDG-17 | An asset opening balance is a snapshot at an effective financial time. Transactions before it remain visible and may contribute to eligible historical budget activity, but are not applied again to the snapshot or subsequent asset balance. Do not infer account balance before the snapshot. When a date-only transaction and opening snapshot share a date and no times establish their ordering, ask the user whether the transaction occurred before or after the snapshot; apply it to subsequent balance only when after. |

This owner-approved product direction changes the prior unqualified asset balance formula; `RULE-BAL-001` is reconciled. The liability opening model has not been changed by this decision. Snapshot-day user choice is owner-approved; behavior when existing transactions or the snapshot are corrected across that boundary remains a review item, not an implied product decision.

## Correction decisions — CR-LDG-003

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. These decisions do not freeze the tactical model.

- **D-LDG-18:** When a corrected transaction moves across an asset opening snapshot, preview the effects; include or exclude its effect from subsequent account balance according to its corrected position, preserve history, and recalculate eligible budget results. Ask for before/after when ordering is unclear under `FR-LDG-012`.
- **D-LDG-19:** If a proposed backdated credit-purchase correction would make a recorded payment exceed recorded debt, surface the conflict and require explicit resolution before saving. Preserve the existing records unless a valid set of changes can be committed together; never silently reduce a real payment or discard its effect.

These choices address transaction correction across a snapshot and corrections invalidating an existing liability payment. The detailed technical mechanism, correction of the snapshot itself, and general historical outstanding behavior remain to be reviewed.

## Opening-snapshot and backdated payment decisions — CR-LDG-004

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. The Ledger tactical model remains a draft.

- **D-LDG-20:** An asset opening snapshot's amount or effective date may be corrected. Before saving, preview the impact on current balance and identify transactions whose relationship to the snapshot changes. Preserve the previous snapshot in audit history and recalculate affected results after confirmation. If a same-day transaction's order relative to the corrected snapshot is unclear, ask the user to select before or after under the existing snapshot rule.
- **D-LDG-21:** Validate a backdated credit-card payment against the recorded debt at the payment's financial time, and check the resulting affected later history. If the operation would create unsupported positive card credit at an affected point, show the conflict and require user correction; do not partially save or silently floor away the conflict. When historical debt or event ordering cannot be determined, surface uncertainty rather than inventing an order. Detailed resolution of ambiguous liability ordering remains to be specified before that case is implemented.

These are approved product behaviors, not a decision about SQL schema or concurrency mechanism. Liability opening-state semantics, same-time ordering and exceptional flows still require review.

## Historical credit-card history decisions — CR-LDG-005

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. The Ledger tactical model is not frozen.

- **D-LDG-22:** If date-only credit-card transactions share a date and their order affects whether a payment is valid, ask the user which occurred first. Preserve the user's ordering choice for deterministic recalculation; do not assume purchases always precede payments. Do not require a choice when the order cannot affect the financial result.
- **D-LDG-23:** If recorded debt immediately before an old payment cannot be determined, show that historical debt is unknown and guide the user to provide or correct the necessary starting information before validating/saving that payment. Do not assume sufficient debt or fabricate an authoritative result. This does not decide that liability opening state follows asset snapshot semantics.

The detailed starting-information workflow, representation and handling of changes invalidating an earlier ordering choice remain technical/domain design work. Product approval is not an independent review.

## Historical balance and ordering revalidation — CR-LDG-006

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Tactical design remains a draft.

- **D-LDG-24:** Provide a guided way to supply an earlier known credit-card debt balance and effective date when needed to validate older history. Identify the later transactions that will be checked and preview impact before saving; preserve prior information in audit history and do not count the same debt twice. The owner subsequently approved the checkpoint relationship and discrepancy behavior in CR-LDG-007.
- **D-LDG-25:** Revalidate affected credit-card history when a transaction changes. Retain an earlier ordering choice only where it still applies; when a new order is needed to determine payment validity, ask the user before saving. Surface conflicts and do not partially save changes.

These choices approve a guided capability and revalidation behavior; they do not approve a multi-snapshot calculation formula, reconciliation algorithm, automatic debt adjustment or liability opening-state semantics.

## Historical card checkpoint reconciliation — CR-LDG-007

**Decision:** Product owner accepted the recommendations in conversation on 2026-11-14. Ledger tactical and persistence design remain subject to review.

- **D-LDG-26:** An earlier known credit-card debt is a historical checkpoint for validating older activity, not an additional debt. Keep the existing later opening balance as the starting point for that point and subsequent balances; never add the earlier checkpoint to it. Compare the earlier balance and attributable intervening transactions against the later balance, using established ordering and valid financial facts.
- **D-LDG-27:** If the comparison does not match, show the discrepancy and guide the user to investigate missing or incorrect records. Do not silently change either balance or create an expense, fee, payment or other transaction. Mark the affected historical calculation incomplete or unavailable until resolved; do not treat that history as fully reconciled or use unknown debt as a valid basis for an old payment. Preserve audit and preview affected changes.

An incomplete or indeterminate comparison is not proof of a numerical mismatch; show uncertainty rather than inventing missing facts. Checkpoint storage, simultaneous edits, same-instant ordering and the exact interface remain technical review items. The approved product outcome does not authorize inferred transactions, alter the separate asset snapshot rules, or declare the tactical specification frozen.

## Equal-time ordering — CR-LDG-008

**Decision:** Product owner selected the recommendation and authorized recommended product decisions for end-of-work review. Recorded 2026-11-18 (Asia/Jakarta). The Ledger tactical design remains under review.

- **D-LDG-28:** When realized financial events have exactly equal explicit timestamps and their order affects a financial result, ask the user for their order and preserve the choice. Do not ask solely to order events when the order cannot change the result. Revalidate the choice if affected records change. If the user cannot establish the required order, show an indeterminate result and do not commit an operation requiring a definite answer; preserve prior canonical state.
- **D-LDG-29:** Apply the same rule when a transaction and an asset opening snapshot or historical card checkpoint share an exact effective timestamp and their relationship affects the result. Ask whether the transaction belongs before or after the balance; do not fabricate a more precise timestamp. Existing same-day date-only rules remain in force.

This does not require invented timestamps, an arbitrary ordering by database identity, or a separate multi-device synchronization design. Ordering choices must be checked for consistency and revalidated when their source records change.

## V1 transfer validation — CR-LDG-009

**Decision:** Product owner authorized the assistant to select recommended product decisions, then explicitly requested freezing the Ledger baseline on 2026-11-18 (Asia/Jakarta). This records owner approval of the selected direction; technical findings remain open.

- **D-LDG-30:** A transfer between the same account is invalid. Explain that the source and destination must be different and suggest an appropriate correction or expense instead if that reflects the real event. Reject the proposed transfer without partial canonical mutation. This does not prevent a truthful expense or an explicit balance adjustment through their own supported flows.

This removes an ambiguity flagged by AUD-LDG-005. It does not define additional account classes, fee behavior or provider-specific logic.

## Owner freeze record — 2026-11-18

The product owner explicitly requested freezing the current Ledger baseline and product decisions. See [Ledger freeze record](../architecture/financial-accounts-ledger-tactical-ddd-draft.md#32-owner-freeze-record). This does not close open findings or imply independent technical review or executable test results.
