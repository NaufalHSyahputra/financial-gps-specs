# Test Scenarios

These are reviewable test specifications, not executed tests. Sources: [financial rules](FINANCIAL_RULES.md), [acceptance catalog](docs/archive/readiness/acceptance-catalog.md), and the linked source IDs. Acceptance cases are draft review targets; amounts in AC cases are IDR. Where a source gives no exact dates, amounts, or action details, choose valid fixtures consistent with the stated behavior; do not treat invented fixtures as approved rules. `BLOCKED` identifies a known open decision or verification boundary; see [DECISIONS.md](DECISIONS.md). Generic edge-case rows may need concrete fixture choices before automation. Cases can appear in multiple slices when their behavior crosses boundaries.


## Slice 1: Financial Calendar & Cycle Boundaries

### EC-CYCLE-001

**Given** Anchor 31 in shorter month.

**When** the described operation is attempted or the state is evaluated.

**Then** Nominal anchor = last calendar day, then apply adjustment policy.

**Source IDs:** EC-CYCLE-001.

### EC-CYCLE-002

**Given** Nominal anchor Saturday, PREVIOUS.

**When** the described operation is attempted or the state is evaluated.

**Then** Walk backward to previous weekday.

**Source IDs:** EC-CYCLE-002.

### EC-CYCLE-003

**Given** Nominal anchor Sunday, NEXT.

**When** the described operation is attempted or the state is evaluated.

**Then** Walk forward to next weekday.

**Source IDs:** EC-CYCLE-003.

### EC-CYCLE-004

**Given** Adjusted next-month anchor crosses month boundary.

**When** the described operation is attempted or the state is evaluated.

**Then** Use resulting effective date; cycle remains previous effective anchor to next effective anchor with no overlap/gap.

**Source IDs:** EC-CYCLE-004.

### EC-CYCLE-005

**Given** Public holiday on weekday.

**When** the described operation is attempted or the state is evaluated.

**Then** Treat as working day in V1 unless holiday mechanism is explicitly added later.

**Source IDs:** EC-CYCLE-005.

### EC-CYCLE-006

**Given** New cycle starts.

**When** the described operation is attempted or the state is evaluated.

**Then** Account/liability/allocation state persists; prior Budgets close; new Budget Spent starts only in newly created instances.

**Source IDs:** EC-CYCLE-006.

### EC-CYCLE-007

**Given** Anchor/policy changes during active cycle.

**When** the described operation is attempted or the state is evaluated.

**Then** Keep the active cycle unchanged. Financial Calendar resolves the transition beginning at its next boundary.

**Source IDs:** EC-CYCLE-007.

### EC-CYCLE-008

**Given** New-anchor transition is under 15 days.

**When** the described operation is attempted or the state is evaluated.

**Then** Do not create a standalone cycle; merge the transition with the following regular cycle as an Extended Cycle.

**Source IDs:** EC-CYCLE-008.

### EC-CYCLE-009

**Given** New-anchor transition is exactly 15 days or longer.

**When** the described operation is attempted or the state is evaluated.

**Then** Create a standalone Short Cycle before regular cycles under the new anchor.

**Source IDs:** EC-CYCLE-009.

### AC-08

**Given** An anchor change is requested mid-cycle; transition to new anchor is 14 days. Repeat with a transition of exactly 15 days.

**When** the stated action is completed or attempted.

**Then** Active cycle remains unchanged; 14-day transition merges into an Extended Cycle; 15-day transition is a standalone Short Cycle.

**Source IDs:** AC-08; FR-CYCLE-001; PRD-CYCLE-001; EC-CYCLE-007; EC-CYCLE-008; EC-CYCLE-009.

### CAL-MATRIX — boundary grid

**Given** anchor values 1 through 31 in every month, leap and non-leap February, and each policy NONE, PREVIOUS_WORKING_DAY, NEXT_WORKING_DAY.

**When** resolve consecutive regular boundaries, including weekend adjustments crossing month/year boundaries.

**Then** clamp before adjustment, and consecutive effective regular boundaries are strictly increasing.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — invalid anchors

**Given** PaydayAnchor = 0 or 32.

**When** construct calendar settings.

**Then** reject the invalid input.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — short-month examples

**Given** anchor 31 in April and February, anchor 30 in February, and anchor 29 in a non-leap February.

**When** resolve the nominal boundary.

**Then** respectively produce 30 April, 28/29 February, 28/29 February, and 28 February before weekday adjustment.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — transition thresholds

**Given** transition lengths of 14, exactly 15, and 16+ days.

**When** change configuration during an active cycle.

**Then** protect the active cycle; classify 14 days EXTENDED, exactly 15 days SHORT, and 16+ days SHORT.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — adjustment before classification

**Given** a nominal 15-day transition becomes less than 15 days after weekend adjustment.

**When** resolve the transition.

**Then** classify the adjusted transition as EXTENDED.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — protected cycle types

**Given** the current cycle is REGULAR, SHORT, or EXTENDED.

**When** request a configuration change.

**Then** preserve that active cycle and protect its boundary.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — pending replace and cancel

**Given** a configuration change is pending but not yet effective.

**When** replace it, or request the current effective settings.

**Then** replace the pending change, or cancel it when the effective settings are requested.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — temporal promotion

**Given** the app remains closed across ProtectedThrough with no background scheduler.

**When** reopen after the pending change becomes effective.

**Then** derive the effective configuration by date without requiring a background job or activation event.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — unnormalized effective change

**Given** a pending configuration has become effective by time but has not been physically normalized.

**When** request another configuration change.

**Then** treat the prior configuration as effective history and protect the then-active cycle.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — initial example

**Given** InitializedAt = 20 Sep and anchor = 10.

**When** initialize the calendar.

**Then** resolve initial cycle 10 Sep through 9 Oct and initial revision EffectiveFrom = 10 Sep.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — historical floor

**Given** an initialized calendar with first revision EffectiveFrom.

**When** resolve the first EffectiveFrom and the preceding date.

**Then** resolve the first date and return CalendarHistoryUnavailable one day before it.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — cycle position

**Given** a valid active cycle.

**When** derive position on the first and last dates and on every in-cycle date.

**Then** first-day Elapsed = 1, final-day Remaining = 1, and Elapsed + Remaining = Total + 1.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — position outside cycle

**Given** an AsOfDate outside the supplied cycle.

**When** derive cycle position.

**Then** reject explicitly without clamping.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.

### CAL-MATRIX — event outcomes

**Given** calendar initialization, schedule, replace, cancel, no-op, elapsed time, and persistence normalization.

**When** perform each operation or let time pass.

**Then** emit initialization and distinct schedule/replace/cancel events; emit no event for no-op, time passage or normalization.

**Source IDs:** Financial Calendar §24; TD-CAL-001–048.


## Slice 2: Accounts, Asset Opening Snapshots & Basic Ledger

**Slice-level implementation blocker — Q-04–06:** Ledger contracts and operation-specific validation remain open for affected paths. Scenario-specific blockers below identify additional known boundaries; an unmarked scenario is not evidence that its Ledger path is ready to implement.

### EC-MONEY-001

**Given** Fraction unsupported by currency minor unit.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject invalid canonical amount; no floating-point storage.

**Source IDs:** EC-MONEY-001.

### EC-TRX-001

**Given** Owned asset transfer.

**When** the described operation is attempted or the state is evaluated.

**Then** No income/expense/Budget Spent.

**Source IDs:** EC-TRX-001.

### EC-TRX-002

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Expense creates Funding Deficit.

**When** the described operation is attempted or the state is evaluated.

**Then** Keep truthful Expense; surface deficit.

**Source IDs:** EC-TRX-002.

### EC-TRX-004

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Void expense.

**When** the described operation is attempted or the state is evaluated.

**Then** Reverse balance and Budget Spent effects; recalc STS/Forecast/Insights.

**Source IDs:** EC-TRX-004.

### EC-REF-001

**Given** Delete record with dependent settlement/reconciliation.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject destructive deletion; use void/cancel/supersede.

**Source IDs:** EC-REF-001.

### EC-LDG-001

**Given** User attempts ordinary new entry against archived account.

**When** the described operation is attempted or the state is evaluated.

**Then** Require restoration first; history remains correctable and eligible archived money still counts.

**Source IDs:** EC-LDG-001.

### EC-LDG-002

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation drawdown.** Baseline expectations below remain review targets.

**Given** Expense exceeds tracked cash balance.

**When** the described operation is attempted or the state is evaluated.

**Then** Save truthful expense; show negative tracked balance; account contributes zero to Available while negative; surface allocation shortfall if present.

**Source IDs:** EC-LDG-002.

### EC-LDG-003

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Payment exceeds recorded liability outstanding.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject without partial mutation and suggest checking missing or incorrect transactions.

**Source IDs:** EC-LDG-003.

### EC-LDG-004

**BLOCKED — AUD-LDG-001 / Q-05: refund consistency and concurrency.** Baseline expectations below remain review targets.

**Given** Partial linked refund.

**When** the described operation is attempted or the state is evaluated.

**Then** Reverse attributable eligible spending only; no duplicate ordinary income.

**Source IDs:** EC-LDG-004.

### EC-LDG-005

**Given** Transfer of Rp100,000 with Rp2,500 fee.

**When** the described operation is attempted or the state is evaluated.

**Then** Transfer principal is not spending; separately recognized fee expense is Rp2,500.

**Source IDs:** EC-LDG-005.

### EC-LDG-006

**Given** Actual transaction predates Calendar historical floor.

**When** the described operation is attempted or the state is evaluated.

**Then** Do not invent a cycle; mark affected cycle-dependent budget results unavailable. Mark affected cycle-based historical reports incomplete or unavailable.

**Source IDs:** EC-LDG-006.

### EC-LDG-007

**Given** Rename/archive category used by historical transaction.

**When** the described operation is attempted or the state is evaluated.

**Then** Preserve reference; show current name and retain rename audit; hide archived category from ordinary new entry.

**Source IDs:** EC-LDG-007.

### EC-LDG-008

**BLOCKED — Q-05 / AUD-LDG-003: dependent correction coordination.** Baseline expectations below remain review targets.

**Given** User corrects or voids linked income.

**When** the described operation is attempted or the state is evaluated.

**Then** Show linked occurrence; obtain appropriate confirmation; preserve audit and update dependent records atomically. On unlink, use EXPECTED before expected time or MISSED after it; if intervening changes make the state unsafe to determine, require guided resolution without partial save.

**Source IDs:** EC-LDG-008.

### EC-LDG-010

**BLOCKED — Q-05 / AUD-LDG-003: dependent correction coordination.** Baseline expectations below remain review targets.

**Given** Transfer with fee is voided but fee really was charged.

**When** the described operation is attempted or the state is evaluated.

**Then** Suggest voiding both; permit keeping the fee after user review, preview effects and atomically commit the selected changes.

**Source IDs:** EC-LDG-010.

### EC-LDG-011

**BLOCKED — Q-05 / AUD-LDG-003: dependent correction coordination.** Baseline expectations below remain review targets.

**Given** Linked income removed after an intervening change makes expected state unsafe to determine.

**When** the described operation is attempted or the state is evaluated.

**Then** Show conflict and require guided resolution; commit no partial correction.

**Source IDs:** EC-LDG-011.

### EC-LDG-015

**Given** Asset opening snapshot Rp1,000,000 on Nov 1; add Rp100,000 expense on Oct 25.

**When** the described operation is attempted or the state is evaluated.

**Then** Retain the expense in history; do not subtract it from the Nov 1 snapshot or subsequent balance again. Historical budget eligibility remains governed by Calendar and Budget.

**Source IDs:** EC-LDG-015.

### EC-LDG-016

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Date-only transaction and asset opening snapshot have same date and no ordering times.

**When** the described operation is attempted or the state is evaluated.

**Then** Ask user whether before or after snapshot; preserve choice; include effect in subsequent balance only if after.

**Source IDs:** EC-LDG-016.

### EC-LDG-017

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Card outstanding Rp60,000; card refund Rp100,000.

**When** the described operation is attempted or the state is evaluated.

**Then** Explain full refund cannot be represented as card credit; do not commit only Rp60,000 or discard Rp40,000; do not record unreceived bank refund.

**Source IDs:** EC-LDG-017.

### EC-LDG-018

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Correct an expense from Nov 2 to Oct 31, crossing a Nov 1 asset opening snapshot.

**When** the described operation is attempted or the state is evaluated.

**Then** Preview the impact; retain history and eligible budget effects but do not subtract the corrected pre-snapshot expense from the snapshot or subsequent balance.

**Source IDs:** EC-LDG-018.

### EC-LDG-020

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Edit an asset opening snapshot's amount or date.

**When** the described operation is attempted or the state is evaluated.

**Then** Preview current-balance impact and transactions whose relationship changes; retain prior snapshot in audit; ask before/after when same-day order is unclear; recalculate after confirmation.

**Source IDs:** EC-LDG-020.

### EC-LDG-031

**Given** User selects the same account as source and destination for a transfer.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject without partial mutation and explain that source and destination must differ.

**Source IDs:** EC-LDG-031.

### AC-02

**Given** Liquid asset A = 1,000,000, asset B = 200,000; transfer 300,000 from A to B.

**When** the stated action is completed or attempted.

**Then** A = 700,000, B = 500,000; total unchanged; no income, expense or Budget Spent. Failure leaves neither leg committed.

**Source IDs:** AC-02; FR-TRX-004; EC-TRX-001; NFR-DATA-001.

### AC-06

**Given** Asset has a non-zero balance and historical activity; user archives it.

**When** the stated action is completed or attempted.

**Then** History and financial effects remain; archived status does not erase its balance.

**Source IDs:** AC-06; FR-ACC-003.

### AC-07

**BLOCKED — Q-05 / AUD-LDG-003: affected expense, allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** A saved eligible expense is voided.

**When** the stated action is completed or attempted.

**Then** Reverse its attributable balance and Budget Spent effects; recalculate applicable STS, forecast and insights; retain audit history. A dependent settlement cannot be destructively deleted.

**Source IDs:** AC-07; FR-TRX-005; EC-TRX-004; EC-REF-001.

### AC-19

**Given** Archive a liquid account holding Rp100,000; attempt a new transaction, then restore it.

**When** the stated action is completed or attempted.

**Then** The eligible balance remains in Available; new ordinary entry requires restoration; historical corrections remain possible.

**Source IDs:** AC-19; FR-LDG-001; EC-LDG-001.

### AC-20

**BLOCKED — Q-05 / AUD-LDG-003: affected expense, allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Cash balance is Rp20,000; record a truthful Rp30,000 expense.

**When** the stated action is completed or attempted.

**Then** Tracked balance is −Rp10,000; display review guidance; its Available contribution is zero and allocation shortfall, if present, is surfaced.

**Source IDs:** AC-20; FR-LDG-002; EC-LDG-002.

### AC-22

**Given** Transfer Rp100,000 and enter an optional Rp2,500 fee.

**When** the stated action is completed or attempted.

**Then** Principal is transfer; fee is a separately recognized expense of Rp2,500. During correction preview both effects; on void suggest voiding both, with the option to retain a fee actually charged; commit selected changes together.

**Source IDs:** AC-22; FR-LDG-008; EC-LDG-005.

### AC-23

**Given** Enter an otherwise valid actual transaction before the Calendar historical floor.

**When** the stated action is completed or attempted.

**Then** Save actual fact; do not invent a cycle; affected cycle-dependent budget results for that date are unavailable. Show the account effect and mark affected cycle-based historical reports incomplete or unavailable.

**Source IDs:** AC-23; FR-LDG-007; EC-LDG-006.

### AC-25

**BLOCKED — Q-05 / AUD-LDG-003: affected expense, allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Correct a Rp1,000,000 bank balance to Rp970,000 using an explained adjustment.

**When** the stated action is completed or attempted.

**Then** Show a separate adjustment, not ordinary income or expense; forecasts begin from corrected actual balance and allocation consistency is recalculated.

**Source IDs:** AC-25; FR-LDG-002; RULE-LDG-003.

### AC-28

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Asset opening snapshot is Rp1,000,000 effective Nov 1; record a Rp100,000 expense from Oct 25.

**When** the stated action is completed or attempted.

**Then** Show the expense in history; the snapshot and later balance are not reduced again. Do not infer pre-snapshot account balance. Historical budget eligibility follows Calendar.

**Source IDs:** AC-28; FR-LDG-012; RULE-BAL-001; EC-LDG-015.

### AC-29

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** A date-only transaction shares the date of an asset opening snapshot, with no times establishing order. Repeat with user selecting before and after.

**When** the stated action is completed or attempted.

**Then** Ask for and preserve the choice. Before does not alter subsequent balance; after applies its effect once. Both remain visible in history.

**Source IDs:** AC-29; FR-LDG-012; RULE-BAL-001; EC-LDG-016.

### AC-31

**BLOCKED — Q-05 / AUD-LDG-003: affected expense, allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Asset snapshot Rp1,000,000 on Nov 1; a Rp100,000 expense originally dated Nov 2 is corrected to Oct 31.

**When** the stated action is completed or attempted.

**Then** Preview change; corrected expense remains in history and no longer reduces the Nov 1 snapshot or subsequent balance; recalculate eligible budgets and preserve audit.

**Source IDs:** AC-31; FR-LDG-012; EC-LDG-018.

### AC-33

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Asset opening snapshot Rp1,000,000 on Nov 1; change it to Rp900,000 or shift date to Nov 2.

**When** the stated action is completed or attempted.

**Then** Preview current-balance effect and transactions whose relationship changes; request ordering where unclear; after confirmation preserve audit and recalculate affected results.

**Source IDs:** AC-33; FR-LDG-012; RULE-BAL-001; EC-LDG-020.

### AC-42

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** An asset expense and its opening snapshot share an exact timestamp; ordering changes the post-snapshot balance.

**When** the stated action is completed or attempted.

**Then** Ask before/after, preserve the user's choice and include its effect only when after; unknown ordering yields indeterminate for a result requiring it.

**Source IDs:** AC-42; FR-LDG-017; RULE-LDG-006; EC-LDG-029.

### AC-44

**Given** Attempt to transfer Rp100,000 from an account to itself.

**When** the stated action is completed or attempted.

**Then** Reject with a useful explanation; account balance, financial facts, budgets and allocations remain unchanged.

**Source IDs:** AC-44; FR-LDG-018; EC-LDG-031.

### LEDGER-13.1

**Given** asset A Rp1,000,000 and asset B Rp200,000.

**When** transfer Rp300,000 from A to B.

**Then** A = Rp700,000 and B = Rp500,000, combined balance unchanged, no budget spending; if destination fails source cannot commit; recalculate Available if eligibility differs.

**Source IDs:** Ledger §13.1; EC-TRX-001.

### LEDGER-MATRIX 22.1-2

**BLOCKED — AUD-LDG-004 / Q-05.** Baseline expectations below remain review targets.

**Given** a correction crosses an opening snapshot, including a date-only same-day case.

**When** preview and correct it.

**Then** compare before/after balances, budget effects and audit; require applicable user choice.

**Source IDs:** Ledger §22.1-2.

### LEDGER-MATRIX 22.1-5

**BLOCKED — AUD-LDG-008.** Baseline expectations below remain review targets.

**Given** an eligible nonzero account is archived.

**When** restart and rebuild Available and allocations.

**Then** retain the account contribution while requiring restoration for ordinary new entry.

**Source IDs:** Ledger §22.1-5.

### LEDGER-MATRIX 22.1-6

**BLOCKED — Q-05 / AUD-LDG-003.** Baseline expectations below remain review targets.

**Given** a transfer has a charged fee.

**When** simulate failure at each mutation, then correct or void selected effects.

**Then** leave no partial update and honor the confirmed fee treatment.

**Source IDs:** Ledger §22.1-6.

### LEDGER-MATRIX 27.6-2

**BLOCKED — AUD-LDG-004 / Q-05.** Baseline expectations below remain review targets.

**Given** a correction was previewed and a dependency changes.

**When** confirm the stale preview.

**Then** reject and refresh it without partial correction.

**Source IDs:** Ledger §27.6-2.


## Slice 3: Allocations, Safe-to-Spend & Daily STS

### EC-STS-001

**Given** Raw STS<0.

**When** the described operation is attempted or the state is evaluated.

**Then** Display STS=0 and Funding Deficit=abs(raw).

**Source IDs:** EC-STS-001.

### EC-STS-002

**Given** Flexible=0, Unallocated>0.

**When** the described operation is attempted or the state is evaluated.

**Then** STS=0; Unallocated is not automatic spend permission.

**Source IDs:** EC-STS-002.

### EC-STS-003

**Given** Locked/Reserved could cover deficit.

**When** the described operation is attempted or the state is evaluated.

**Then** Do not automatically release.

**Source IDs:** EC-STS-003.

### EC-STS-004

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Allocation shortfall.

**When** the described operation is attempted or the state is evaluated.

**Then** Authoritative STS/Daily STS unresolved.

**Source IDs:** EC-STS-004.

### EC-DSTS-001

**Given** Final cycle day.

**When** the described operation is attempted or the state is evaluated.

**Then** Remaining days=1.

**Source IDs:** EC-DSTS-001.

### EC-STS-BND-001

**Given** Obligation due final day of active cycle.

**When** the described operation is attempted or the state is evaluated.

**Then** It is STS-applicable.

**Source IDs:** EC-STS-BND-001.

### EC-STS-BND-002

**Given** Obligation due one day after active cycle.

**When** the described operation is attempted or the state is evaluated.

**Then** It does not reduce current STS; Planning/Forecast evaluates future risk.

**Source IDs:** EC-STS-BND-002.

### EC-OBL-001

**Given** Overdue unpaid obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Remains STS-applicable and visible.

**Source IDs:** EC-OBL-001.

### EC-OBL-002

**Given** Partial payment.

**When** the described operation is attempted or the state is evaluated.

**Then** Remaining due stays active.

**Source IDs:** EC-OBL-002.

### EC-OBL-003

**Given** Reservation fully covers obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Flexible STS deduction for that obligation=0.

**Source IDs:** EC-OBL-003.

### EC-OBL-004

**Given** Reservation partially covers applicable obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Deduct only `max(0, Applicable Obligation Amount - Applicable Reserved Coverage)` from Flexible.

**Source IDs:** EC-OBL-004.

### EC-REC-001

**Given** Staged recurring periods overlap.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject rule/stage edit.

**Source IDs:** EC-REC-001.

### EC-REC-002

**Given** Stage boundary date.

**When** the described operation is attempted or the state is evaluated.

**Then** Occurrence uses stage whose effective interval contains occurrence date; intervals must be unambiguous.

**Source IDs:** EC-REC-002.

### EC-ALLOC-NEW-001

**Given** Starting balance created during onboarding.

**When** the described operation is attempted or the state is evaluated.

**Then** Newly available amount is entirely Unallocated unless explicitly allocated; STS does not increase from Unallocated.

**Source IDs:** EC-ALLOC-NEW-001.

### EC-ALLOC-NEW-002

**Given** Salary received.

**When** the described operation is attempted or the state is evaluated.

**Then** Actual balance/Available increase; new delta remains Unallocated until explicit allocation.

**Source IDs:** EC-ALLOC-NEW-002.

### EC-ALLOC-NEW-003

**Given** Freelance income received.

**When** the described operation is attempted or the state is evaluated.

**Then** Actual balance/Available increase; new delta remains Unallocated until explicit allocation.

**Source IDs:** EC-ALLOC-NEW-003.

### EC-ALLOC-NEW-004

**Given** User partially allocates newly available money.

**When** the described operation is attempted or the state is evaluated.

**Then** Allocated portion moves to selected role; remainder stays Unallocated.

**Source IDs:** EC-ALLOC-NEW-004.

### EC-ALLOC-NEW-005

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** User reallocates Flexible/Reserved/Locked.

**When** the described operation is attempted or the state is evaluated.

**Then** Apply explicit atomic allocation change; conservation must hold and no implicit destination is inferred.

**Source IDs:** EC-ALLOC-NEW-005.

### EC-TRX-003

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Expense causes allocations > Available.

**When** the described operation is attempted or the state is evaluated.

**Then** Keep Expense; create allocation shortfall; STS unresolved until explicit resolution.

**Source IDs:** EC-TRX-003.

### EC-CONC-001

**BLOCKED — Q-05 / AUD-LDG-003: concurrent allocation consistency.** Baseline expectations below remain review targets.

**Given** Concurrent allocation edits oversubscribe.

**When** the described operation is attempted or the state is evaluated.

**Then** Serialize/reject one commit; never silently violate normal conservation.

**Source IDs:** EC-CONC-001.

### AC-01

**Given** New user selects an Indonesian timezone, cycle settings and starting Cash of 1,000,000; finishes onboarding without adding another account or allocating.

**When** the stated action is completed or attempted.

**Then** Cash/Available = 1,000,000; Unallocated = 1,000,000; Flexible = 0; STS = 0 absent obligations.

**Source IDs:** AC-01; FR-ONB-001; EC-ALLOC-NEW-001; EC-STS-002.

### AC-05

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation drawdown.** Baseline expectations below remain review targets.

**Given** Available 1,000,000, with 800,000 Flexible and 200,000 Reserved. A truthful asset expense reduces Available by 300,000.

**When** the stated action is completed or attempted.

**Then** Record remains; allocations exceed Available by 300,000; surface exact shortfall and unresolved STS/Daily STS until explicit resolution. Do not silently release intent.

**Source IDs:** AC-05; FR-ALLOC-003; EC-TRX-003; EC-STS-004.

### AC-09

**Given** Flexible 3,800,000; applicable obligation 1,550,000 with applicable Reserved coverage 500,000; no shortfall.

**When** the stated action is completed or attempted.

**Then** Unfunded obligation = 1,050,000; STS = 2,750,000. Expected salary does not increase it.

**Source IDs:** AC-09; FR-STS-001; FR-OBL-003; EC-OBL-004; PRD-STS-001.

### AC-15

**Given** Obligation 1,000,000 due this cycle, partially paid 300,000, with 200,000 applicable Reserved coverage.

**When** the stated action is completed or attempted.

**Then** Remaining due 700,000; uncovered portion 500,000. If unpaid overdue it remains visible and STS-applicable.

**Source IDs:** AC-15; FR-OBL-002; FR-OBL-003; EC-OBL-001; EC-OBL-002.

### AC-18

**Given** User enters overlapping stages for a recurring obligation.

**When** the stated action is completed or attempted.

**Then** Reject overlapping edit and leave existing rule unchanged.

**Source IDs:** AC-18; FR-REC-002; EC-REC-001; NFR-DATA-002.

### EC-NUM-001

**Given** Flexible actual Rp5,000,000, expected salary Rp10,000,000 and current-cycle obligation Rp6,000,000.

**When** calculate STS.

**Then** raw STS = -Rp1,000,000, displayed STS = Rp0 and funding deficit = Rp1,000,000; salary affects forecast only.

**Source IDs:** EC-NUM-001.

### RULE-STS-101 example

**Given** Available Rp6,800,000; Flexible Rp3,800,000; Reserved Rp1,000,000; Locked Rp2,000,000; applicable obligation Rp1,550,000; linked Reserved Rp500,000; expected salary Rp10,000,000 tomorrow.

**When** calculate current STS.

**Then** unfunded obligation = Rp1,050,000 and raw STS = Rp2,750,000; expected salary contributes Rp0.

**Source IDs:** RULE-STS-101.

### RULE-ALLOC-101A example

**Given** starting Cash Rp5,000,000 with no allocations.

**When** calculate allocation and STS.

**Then** Flexible = Rp0, Reserved = Rp0, Locked = Rp0, Unallocated = Rp5,000,000, STS = Rp0.

**Source IDs:** RULE-ALLOC-101A.

### LEDGER-13.4

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation drawdown.** Baseline expectations below remain review targets.

**Given** Available Rp1,000,000, Flexible Rp800,000 and Reserved Rp200,000.

**When** record a truthful Rp300,000 asset expense.

**Then** Ledger example says Available Rp700,000 and unchanged allocations Rp1,000,000 yield shortfall Rp300,000 and unresolved STS/Daily STS; do not silently reduce an allocation.

**Source IDs:** Ledger §13.4; RULE-ALLOC-102; RULE-BUD-105.

### RULE-REC-102 staged example

**Given** stages 2026-10-01..2028-09-30 Rp3,200,000/month; 2028-10-01..2030-09-30 Rp3,750,000/month; 2030-10-01..open Rp4,100,000/month.

**When** generate occurrences on either side of each boundary.

**Then** resolve exactly the one stage containing each occurrence date.

**Source IDs:** RULE-REC-102; EC-REC-002.


## Slice 4: Budgets, Pace & Run-Rate Projections

### EC-BUD-001

**Given** Budget Amount=0, Spent=0.

**When** the described operation is attempted or the state is evaluated.

**Then** Utilization=0%, no overspend.

**Source IDs:** EC-BUD-001.

### EC-BUD-002

**Given** Budget Amount=0, actual expense >0.

**When** the described operation is attempted or the state is evaluated.

**Then** `OVERSPENT_UNBUDGETED`; do not display infinite percentage.

**Source IDs:** EC-BUD-002.

### EC-BUD-003

**Given** Utilization exactly 80%.

**When** the described operation is attempted or the state is evaluated.

**Then** `APPROACHING_LIMIT`.

**Source IDs:** EC-BUD-003.

### EC-BUD-004

**Given** Utilization exactly 100%.

**When** the described operation is attempted or the state is evaluated.

**Then** `LIMIT_REACHED`.

**Source IDs:** EC-BUD-004.

### EC-BUD-005

**Given** Utilization >100%.

**When** the described operation is attempted or the state is evaluated.

**Then** `OVERSPENT`, show overspend amount.

**Source IDs:** EC-BUD-005.

### EC-BUD-006

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Expense maps to budget.

**When** the described operation is attempted or the state is evaluated.

**Then** Expense reduces actual money once and increases Budget Spent; budget does not deduct cash again.

**Source IDs:** EC-BUD-006.

### EC-BUD-007

**Given** Unused budget at cycle close.

**When** the described operation is attempted or the state is evaluated.

**Then** Does not create money. Existing unspent Flexible remains; old budget closes.

**Source IDs:** EC-BUD-007.

### EC-BUD-008

**Given** Apply previous-cycle template.

**When** the described operation is attempted or the state is evaluated.

**Then** Create new Budget IDs, same proposed definitions after user confirmation, Spent=0.

**Source IDs:** EC-BUD-008.

### EC-BUD-009

**Given** Refund reverses prior eligible expense.

**When** the described operation is attempted or the state is evaluated.

**Then** Reduce affected Budget Spent up to attributable reversed spending; do not create duplicate income semantics.

**Source IDs:** EC-BUD-009.

### EC-BUD-010

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Expense recategorized.

**When** the described operation is attempted or the state is evaluated.

**Then** Remove from old mapped budget and add to new mapped budget for same effective cycle.

**Source IDs:** EC-BUD-010.

### EC-PACE-001

**Given** Consumption exactly cycle progress+5pp.

**When** the described operation is attempted or the state is evaluated.

**Then** `ON_PACE`; only `>` tolerance is ahead.

**Source IDs:** EC-PACE-001.

### EC-PROJ-001

**Given** Day 1 or Day 2 of cycle.

**When** the described operation is attempted or the state is evaluated.

**Then** Utilization/progress/pace may be shown; numeric Projected Spending/Overspend is `NOT_YET_ELIGIBLE`.

**Source IDs:** EC-PROJ-001.

### EC-PROJ-001A

**Given** Day 3 with Budget Spent=0.

**When** the described operation is attempted or the state is evaluated.

**Then** Projection becomes eligible; Projected Spending=0 and Projected Overspend=0.

**Source IDs:** EC-PROJ-001A.

### EC-PROJ-002

**Given** Transaction history known incomplete.

**When** the described operation is attempted or the state is evaluated.

**Then** Projection=`INSUFFICIENT_DATA`; no numeric invented projection.

**Source IDs:** EC-PROJ-002.

### EC-PROJ-003

**Given** Already overspent.

**When** the described operation is attempted or the state is evaluated.

**Then** Daily amount to stay within original budget=0; explain overspend cannot be undone.

**Source IDs:** EC-PROJ-003.

### EC-BUD-CAP-001

**Given** Total Active Budget < Flexible.

**When** the described operation is attempted or the state is evaluated.

**Then** Valid; no budget-capacity warning.

**Source IDs:** EC-BUD-CAP-001.

### EC-BUD-CAP-002

**Given** Total Active Budget = Flexible.

**When** the described operation is attempted or the state is evaluated.

**Then** Valid; no budget-capacity warning.

**Source IDs:** EC-BUD-CAP-002.

### EC-BUD-CAP-003

**Given** Total Active Budget > Flexible.

**When** the described operation is attempted or the state is evaluated.

**Then** Valid; do not block; show `BUDGET_EXCEEDS_FLEXIBLE` by exact difference. No money is reserved.

**Source IDs:** EC-BUD-CAP-003.

### EC-BUD-CAP-004

**Given** Budget created/edited/deleted.

**When** the described operation is attempted or the state is evaluated.

**Then** Flexible/Reserved/Locked/Unallocated/Available do not change solely because of Budget mutation.

**Source IDs:** EC-BUD-CAP-004.

### EC-PROJ-004

**Given** Budget created after cycle Day 3.

**When** the described operation is attempted or the state is evaluated.

**Then** Projection is eligible immediately using cycle ElapsedCycleDays and all eligible mapped cycle spending.

**Source IDs:** EC-PROJ-004.

### EC-PROJ-005

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Backdated eligible expense.

**When** the described operation is attempted or the state is evaluated.

**Then** Recompute Budget Spent, utilization, pace, projection/overspend if eligible, and Insights.

**Source IDs:** EC-PROJ-005.

### EC-PROJ-006

**BLOCKED — Q-05 / AUD-LDG-003: expense allocation or correction coordination.** Baseline expectations below remain review targets.

**Given** Deleted/voided eligible expense.

**When** the described operation is attempted or the state is evaluated.

**Then** Remove its budget contribution and recompute projection/insights.

**Source IDs:** EC-PROJ-006.

### AC-10

**Given** Flexible 1,000,000 and active budget totals 1,000,000; create another budget of 200,000.

**When** the stated action is completed or attempted.

**Then** Save succeeds; warning difference = 200,000; Available and allocation roles do not change.

**Source IDs:** AC-10; FR-BUD-001; EC-BUD-CAP-003; EC-BUD-CAP-004.

### AC-11

**Given** Budget on cycle day 2 with complete history; view projection, then view on day 3 with zero spent.

**When** the stated action is completed or attempted.

**Then** Day 2 numeric projection unavailable; day 3 projected spending and overspend are zero. Incomplete history shows INSUFFICIENT_DATA instead.

**Source IDs:** AC-11; FR-BUD-005; EC-PROJ-001; EC-PROJ-001A; EC-PROJ-002.

### AC-12

**Given** At cycle close a budget has unspent amount; user previews and applies it as a template in the new cycle.

**When** the stated action is completed or attempted.

**Then** Initially no new-cycle budget; applying creates new instances with Spent=0. No budget rollover creates money and actual allocations persist.

**Source IDs:** AC-12; FR-BUD-006; FR-BUD-007; EC-BUD-007; EC-BUD-008.

### EC-NUM-002

**Given** 30 cycle days, 18 elapsed days, budget Rp1,500,000, spent Rp1,280,000.

**When** calculate run rate and remaining daily budget.

**Then** average/day Rp71,111.11..., projected spending Rp2,133,333, projected overspend Rp633,333, future days 12, remaining Rp220,000, maximum average/day Rp18,333.

**Source IDs:** EC-NUM-002.

### EC-NUM-003

**Given** old budget Rp1,500,000, old spent Rp1,200,000, unused budget Rp300,000.

**When** close the cycle and use the prior budget as a template.

**Then** create no Rp300,000 transaction; unspent cash remains Flexible if it was Flexible; create a new Rp1,500,000 budget with Spent=0.

**Source IDs:** EC-NUM-003.


## Slice 5: Expected Income, Forecast & Funding Dependency

### EC-INC-001

**Given** Expected salary before obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Salary contributes zero to current STS; may improve Forecast only.

**Source IDs:** EC-INC-001.

### EC-INC-002

**Given** Uncertain freelance income.

**When** the described operation is attempted or the state is evaluated.

**Then** Base forecast excludes it; scenario may include it and must label assumption.

**Source IDs:** EC-INC-002.

### EC-INC-003

**Given** Actual income arrives early/late/with a different amount and user explicitly links it.

**When** the described operation is attempted or the state is evaluated.

**Then** Posted actual Income is authoritative; mark the Expected Income `RECEIVED` without duplicate inflow.

**Source IDs:** EC-INC-003.

### EC-INC-004

**Given** Expected income missed/cancelled.

**When** the described operation is attempted or the state is evaluated.

**Then** Remove future forecast event; current actual STS was never dependent on it.

**Source IDs:** EC-INC-004.

### EC-INC-005

**Given** Similar actual Income exists but user does not link it.

**When** the described operation is attempted or the state is evaluated.

**Then** Keep actual Income valid and unlinked; do not mark Expected Income `RECEIVED` from amount/date/description/source similarity.

**Source IDs:** EC-INC-005.

### EC-INC-006

**Given** Late actual receipt is explicitly linked to a `MISSED` occurrence.

**When** the described operation is attempted or the state is evaluated.

**Then** Transition `MISSED` to `RECEIVED`; actual amount/time is authoritative.

**Source IDs:** EC-INC-006.

### EC-ADV-001

**Given** Advisor wants numeric corrective amount.

**When** the described operation is attempted or the state is evaluated.

**Then** Must use engine output; omit amount if required input/projection unavailable.

**Source IDs:** EC-ADV-001.

### EC-FDEP-001

**Given** Post-cycle obligation due before supporting expected income.

**When** the described operation is attempted or the state is evaluated.

**Then** No Funding Dependency to that later income; surface projected deficit/risk if actual-only funds are insufficient.

**Source IDs:** EC-FDEP-001.

### EC-FDEP-002

**Given** Post-cycle obligation due after EXPECTED_STABLE income and actual-only funds insufficient.

**When** the described operation is attempted or the state is evaluated.

**Then** Funding Dependency exists; current STS remains unchanged.

**Source IDs:** EC-FDEP-002.

### EC-FDEP-003

**Given** Supporting EXPECTED_STABLE income becomes MISSED or moves after obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Remove dependency-to-income assumption and surface projected funding deficit/risk.

**Source IDs:** EC-FDEP-003.

### EC-FDEP-004

**Given** Supporting expected income never arrives.

**When** the described operation is attempted or the state is evaluated.

**Then** It never becomes Actual; after expected time it is MISSED and forecast/funding risk recalculates.

**Source IDs:** EC-FDEP-004.

### EC-FDEP-005

**Given** Sufficient actual funding becomes available before obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Funding Dependency disappears after recalculation.

**Source IDs:** EC-FDEP-005.

### EC-FCST-BASE-001

**Given** EXPECTED_STABLE future income.

**When** the described operation is attempted or the state is evaluated.

**Then** Included in Base Forecast, never in current STS.

**Source IDs:** EC-FCST-BASE-001.

### EC-FCST-BASE-002

**Given** EXPECTED_UNCERTAIN future income.

**When** the described operation is attempted or the state is evaluated.

**Then** Excluded from Base Forecast.

**Source IDs:** EC-FCST-BASE-002.

### EC-FCST-SCN-001

**Given** User selects uncertain income scenario.

**When** the described operation is attempted or the state is evaluated.

**Then** Add selected uncertain event to Scenario Forecast only; Actual and Base remain unchanged.

**Source IDs:** EC-FCST-SCN-001.

### EC-FCST-RECV-001

**Given** Expected Income explicitly linked to received actual Income.

**When** the described operation is attempted or the state is evaluated.

**Then** Mark Expected Income `RECEIVED`; actual amount/time authoritative; newly available delta remains Unallocated.

**Source IDs:** EC-FCST-RECV-001.

### EC-FCST-LATE-001

**Given** Expected time passes without receipt.

**When** the described operation is attempted or the state is evaluated.

**Then** Expected Income becomes MISSED; never silently becomes actual.

**Source IDs:** EC-FCST-LATE-001.

### AC-03

**Given** Asset 1,000,000; actual unlinked income 500,000 is recorded.

**When** the stated action is completed or attempted.

**Then** Asset increases by 500,000; newly available delta is Unallocated; existing explicit allocations remain; no expected occurrence is implicitly received.

**Source IDs:** AC-03; FR-TRX-002; FR-INC-003; EC-INC-005.

### AC-13

**Given** Create stable salary and uncertain freelance future income; view base and a scenario selecting freelance.

**When** the stated action is completed or attempted.

**Then** Stable income appears in Base; uncertain only in selected scenario. Neither changes actual balance, Flexible or STS.

**Source IDs:** AC-13; FR-INC-002; FR-INC-004; EC-FCST-BASE-001; EC-FCST-SCN-001.

### AC-14

**Given** Expected occurrence becomes MISSED; later user posts an actual income with a different amount and explicitly links it.

**When** the stated action is completed or attempted.

**Then** Occurrence becomes RECEIVED; actual amount/time govern; no duplicate inflow; newly available money remains Unallocated. Without explicit link it remains unlinked.

**Source IDs:** AC-14; FR-INC-003; FR-INC-005; EC-INC-006; EC-FCST-RECV-001.

### AC-16

**Given** Post-cycle obligation lacks actual-only projected funding; stable expected income is scheduled before it. Then that income becomes MISSED.

**When** the stated action is completed or attempted.

**Then** Show Funding Dependency first; after missed income recalculate and show projected funding risk/deficit. Current-cycle STS horizon is unchanged.

**Source IDs:** AC-16; FR-FCST-004; EC-FDEP-002; EC-FDEP-003; EC-STS-BND-002.

### AC-17

**Given** An insight requires a numeric projection but projection inputs are unavailable.

**When** the stated action is completed or attempted.

**Then** Explain what is known; omit the unavailable numeric recommendation and do not invent a value.

**Source IDs:** AC-17; FR-ADV-003; EC-ADV-001.

### AC-26

**BLOCKED — Q-05 / AUD-LDG-003: income unlink and correction coordination.** Baseline expectations below remain review targets.

**Given** Unlink actual income before expected time; repeat after expected time has passed.

**When** the stated action is completed or attempted.

**Then** First occurrence becomes EXPECTED; second becomes MISSED. If intervening lifecycle changes make the correct state unsafe to determine, require guided resolution without a partial save.

**Source IDs:** AC-26; FR-LDG-003; EC-LDG-008; EC-LDG-011.

### LEDGER-13.3

**BLOCKED — AUD-LDG-006 / Q-05: unlinked-income regression and link coordination.** Baseline expectations below remain review targets.

**Given** expected salary Rp5,000,000 and posted actual income Rp4,800,000.

**When** record income without a link, then explicitly link it.

**Then** increase asset by Rp4,800,000; do not infer receipt without a link; valid link creates no duplicate inflow; newly available money remains Unallocated.

**Source IDs:** Ledger §13.3; EC-INC-005.

### LEDGER-MATRIX 22.1-4

**BLOCKED — Q-05 / AUD-LDG-003.** Baseline expectations below remain review targets.

**Given** explicitly linked salary is voided before and after expected time.

**When** confirm the void.

**Then** derive EXPECTED/MISSED; do not partially write an unsafe intervening lifecycle change or duplicate a forecast event.

**Source IDs:** Ledger §22.1-4.


## Slice 6: Credit Cards, BNPL, Installments, Refunds & Historical Checkpoints

**Slice-level implementation blocker — Q-04–06:** Ledger contracts and operation-specific validation remain open for affected paths. Scenario-specific blockers below identify additional known boundaries; an unmarked scenario is not evidence that its Ledger path is ready to implement.

### EC-CC-001

**Given** Card purchase.

**When** the described operation is attempted or the state is evaluated.

**Then** Expense recognized at purchase; liability increases.

**Source IDs:** EC-CC-001.

### EC-CC-002

**Given** Card payment.

**When** the described operation is attempted or the state is evaluated.

**Then** Asset/liability decrease; no second category expense.

**Source IDs:** EC-CC-002.

### EC-CC-003

**Given** Purchase moved across cutoff by correction.

**When** the described operation is attempted or the state is evaluated.

**Then** Reassign statement/obligation atomically; purchase expense remains once.

**Source IDs:** EC-CC-003.

### EC-INST-001

**Given** Installment includes fees/interest.

**When** the described operation is attempted or the state is evaluated.

**Then** Use explicit actual installment schedule, not principal/count assumption.

**Source IDs:** EC-INST-001.

### EC-INST-002

**Given** Conversion would duplicate original repayment obligation.

**When** the described operation is attempted or the state is evaluated.

**Then** Supersede original future repayment representation atomically.

**Source IDs:** EC-INST-002.

### EC-BNPL-001

**Given** Rp800k purchase + Rp25k fee.

**When** the described operation is attempted or the state is evaluated.

**Then** Recognize purchase expense and explicit fee expense/liability per timing rule; total liability Rp825k when both incurred; repayment is not duplicate purchase expense.

**Source IDs:** EC-BNPL-001.

### EC-LDG-003

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Payment exceeds recorded liability outstanding.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject without partial mutation and suggest checking missing or incorrect transactions.

**Source IDs:** EC-LDG-003.

### EC-LDG-004

**BLOCKED — AUD-LDG-001 / Q-05: refund consistency and concurrency.** Baseline expectations below remain review targets.

**Given** Partial linked refund.

**When** the described operation is attempted or the state is evaluated.

**Then** Reverse attributable eligible spending only; no duplicate ordinary income.

**Source IDs:** EC-LDG-004.

### EC-LDG-009

**BLOCKED — AUD-LDG-001 / Q-05: refund consistency and concurrency.** Baseline expectations below remain review targets.

**Given** Unlinked refund later linked to original purchase.

**When** the described operation is attempted or the state is evaluated.

**Then** Before linking, increase receiving account and report separately without automatic budget change. After user confirms the link, recalculate attributable eligible spending with audit; no duplicate inflow.

**Source IDs:** EC-LDG-009.

### EC-LDG-012

**BLOCKED — AUD-LDG-001 / Q-05: refund consistency and concurrency.** Baseline expectations below remain review targets.

**Given** Purchase Rp500,000 already has Rp300,000 linked refunds; attempt to link another Rp250,000.

**When** the described operation is attempted or the state is evaluated.

**Then** Reject the link, explain Rp200,000 remains; no partial mutation.

**Source IDs:** EC-LDG-012.

### EC-LDG-013

**BLOCKED — AUD-LDG-001 / Q-05: refund consistency and concurrency.** Baseline expectations below remain review targets.

**Given** Edit a purchase to Rp200,000 when Rp300,000 of valid refunds are linked.

**When** the described operation is attempted or the state is evaluated.

**Then** Show conflict; require explicit resolution; do not partially commit or silently alter refunds.

**Source IDs:** EC-LDG-013.

### EC-LDG-014

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Refund to fully paid card would produce positive card credit.

**When** the described operation is attempted or the state is evaluated.

**Then** Explain unsupported representation; do not drop the credit or record an unreceived bank inflow.

**Source IDs:** EC-LDG-014.

### EC-LDG-017

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Card outstanding Rp60,000; card refund Rp100,000.

**When** the described operation is attempted or the state is evaluated.

**Then** Explain full refund cannot be represented as card credit; do not commit only Rp60,000 or discard Rp40,000; do not record unreceived bank refund.

**Source IDs:** EC-LDG-017.

### EC-LDG-019

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Correct a Rp500,000 credit purchase to Rp300,000 when a Rp500,000 payment exists.

**When** the described operation is attempted or the state is evaluated.

**Then** Show payment conflict; require explicit resolution before saving; preserve prior facts until a valid set of changes commits atomically.

**Source IDs:** EC-LDG-019.

### EC-LDG-021

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Credit-card debt Rp300,000 on Nov 5; Rp200,000 purchase Nov 10; enter Rp400,000 payment dated Nov 5.

**When** the described operation is attempted or the state is evaluated.

**Then** Validate debt at payment time and affected later history; show conflict and require correction; no partial mutation or silently inferred ordering.

**Source IDs:** EC-LDG-021.

### EC-LDG-022

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Same-day date-only card purchase and payment can be valid in one order and invalid in another.

**When** the described operation is attempted or the state is evaluated.

**Then** Ask which occurred first, preserve choice, and validate against that order; no automatic guess or partial write.

**Source IDs:** EC-LDG-022.

### EC-LDG-023

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Recorded card debt immediately before an old payment is unknown.

**When** the described operation is attempted or the state is evaluated.

**Then** Show unknown history and guide user to provide or correct needed starting information before validation/save. Do not assume sufficient debt.

**Source IDs:** EC-LDG-023.

### EC-LDG-024

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Historical card debt is unknown and user provides an earlier known balance.

**When** the described operation is attempted or the state is evaluated.

**Then** Identify affected later transactions, preview impact and preserve prior information in audit; do not count debt twice. Compare against the later opening balance without adding the two; flag mismatch or indeterminate history.

**Source IDs:** EC-LDG-024.

### EC-LDG-025

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Editing a card transaction invalidates a previous same-day ordering choice.

**When** the described operation is attempted or the state is evaluated.

**Then** Revalidate affected history; retain the choice only if applicable, ask again if needed for payment validity, and prevent partial save on conflict.

**Source IDs:** EC-LDG-025.

### EC-LDG-026

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Earlier card debt Rp700,000 plus recorded intervening effects yields Rp950,000 at a later opening of Rp1,000,000.

**When** the described operation is attempted or the state is evaluated.

**Then** Show Rp50,000 discrepancy; retain both recorded amounts, do not invent a transaction, and mark affected historical calculation incomplete/unavailable until resolved.

**Source IDs:** EC-LDG-026.

### EC-LDG-027

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Earlier card checkpoint and later opening lack sufficient intervening history or ordering for a reliable comparison.

**When** the described operation is attempted or the state is evaluated.

**Then** Show comparison as indeterminate; do not invent an exact difference or assume sufficient debt to validate an old payment.

**Source IDs:** EC-LDG-027.

### EC-LDG-028

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Card purchase and payment have the same explicit timestamp and order changes payment validity.

**When** the described operation is attempted or the state is evaluated.

**Then** Ask for and preserve the user's order; if unknown, return indeterminate and do not partially commit the operation requiring a definite result.

**Source IDs:** EC-LDG-028.

### EC-LDG-029

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Transaction and opening snapshot or historical checkpoint share an exact timestamp and the result depends on their order.

**When** the described operation is attempted or the state is evaluated.

**Then** Ask whether transaction is before or after the balance, preserve choice; if unknown return indeterminate.

**Source IDs:** EC-LDG-029.

### EC-LDG-030

**BLOCKED — AUD-LDG-002 / Q-05: historical liability or ordering boundary.** Baseline expectations below remain review targets.

**Given** Equal-time events produce the same result in every applicable order.

**When** the described operation is attempted or the state is evaluated.

**Then** Do not ask for ordering solely for the tie; retain deterministic financial result.

**Source IDs:** EC-LDG-030.

### AC-04

**Given** User records a credit purchase of 800,000, then pays the liability from an asset account.

**When** the stated action is completed or attempted.

**Then** Purchase produces one expense and increases liability; payment reduces asset and liability and creates no second purchase expense.

**Source IDs:** AC-04; FR-CRD-001; FR-CRD-003; EC-CC-001; EC-CC-002.

### AC-21

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Liability outstanding is Rp1,000,000; attempt a new payment of Rp1,100,000.

**When** the stated action is completed or attempted.

**Then** Reject with guidance; no partial asset, liability or settlement mutation.

**Source IDs:** AC-21; FR-LDG-004; EC-LDG-003.

### AC-24

**BLOCKED — AUD-LDG-001 / Q-05: refund coordination and concurrency verification.** Baseline expectations below remain review targets.

**Given** Record an unlinked refund of Rp100,000, then confirm a link to its original eligible purchase.

**When** the stated action is completed or attempted.

**Then** Initially increase the receiving account and report the refund separately without a budget change. On linking, recalculate attributable budget spending, preserve audit and do not add a second inflow.

**Source IDs:** AC-24; FR-LDG-005; EC-LDG-009.

### AC-27

**BLOCKED — AUD-LDG-001 / Q-05: refund coordination and concurrency verification.** Baseline expectations below remain review targets.

**Given** A purchase for Rp500,000 already has Rp300,000 of valid linked refunds; attempt to link another Rp250,000.

**When** the stated action is completed or attempted.

**Then** Reject the proposed link and show Rp200,000 remains; existing facts and links remain unchanged. Editing purchase below valid linked refunds requires explicit resolution before commit.

**Source IDs:** AC-27; FR-LDG-009; EC-LDG-012; EC-LDG-013.

### AC-30

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Card outstanding is Rp60,000; attempt a Rp100,000 refund to the card.

**When** the stated action is completed or attempted.

**Then** Explain the full refund is unsupported; do not commit Rp60,000 as a partial refund or discard Rp40,000, and do not record bank money not received.

**Source IDs:** AC-30; FR-LDG-011; RULE-LDG-002; EC-LDG-017.

### AC-32

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** A Rp500,000 credit purchase and Rp500,000 payment exist; attempt to correct purchase to Rp300,000.

**When** the stated action is completed or attempted.

**Then** Show conflict and require explicit resolution; do not silently reduce payment or commit a partial change.

**Source IDs:** AC-32; FR-LDG-013; EC-LDG-019.

### AC-34

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Recorded credit-card debt is Rp300,000 on Nov 5; a Rp200,000 purchase occurs Nov 10; user records Rp400,000 payment dated Nov 5.

**When** the stated action is completed or attempted.

**Then** Show unsupported-credit conflict at payment time; do not commit a partial payment or silently rely on later purchase. Indeterminate ordering is surfaced rather than guessed.

**Source IDs:** AC-34; FR-LDG-014; RULE-LDG-002; EC-LDG-021.

### AC-35

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Same-day date-only card purchase Rp500,000 and payment Rp500,000; only purchase-first makes the payment valid.

**When** the stated action is completed or attempted.

**Then** Ask which occurred first and preserve that order. Validate accordingly; if invalid, surface conflict without partial write. If order cannot affect validity, do not ask.

**Source IDs:** AC-35; FR-LDG-014; EC-LDG-022.

### AC-36

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Enter an October card payment but recorded debt immediately before it cannot be established.

**When** the stated action is completed or attempted.

**Then** Show historical debt as unknown; guide the user to provide or correct needed starting information before validation/save; do not assume debt was sufficient.

**Source IDs:** AC-36; FR-LDG-014; EC-LDG-023.

### AC-37

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** An old card payment cannot be validated; user enters an earlier known debt and effective date.

**When** the stated action is completed or attempted.

**Then** Show the later transactions to be checked and preview impact; preserve audit and do not count debt twice. Treat the earlier balance as a checkpoint; compare with the later opening without adding the balances, and surface a mismatch or indeterminate history.

**Source IDs:** AC-37; FR-LDG-015; EC-LDG-024.

### AC-38

**BLOCKED — AUD-LDG-004 / Q-04: snapshot as-of and ordering revalidation.** Baseline expectations below remain review targets.

**Given** Edit a card transaction so a prior same-day ordering choice no longer applies and ordering affects payment validity.

**When** the stated action is completed or attempted.

**Then** Revalidate affected history; ask for the required new ordering; on conflict or cancellation preserve existing records without partial save.

**Source IDs:** AC-38; FR-LDG-016; EC-LDG-025.

### AC-39

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Earlier card debt Rp700,000 plus determinable intervening effects gives Rp950,000 at a later opening recorded as Rp1,000,000.

**When** the stated action is completed or attempted.

**Then** Show a Rp50,000 discrepancy; preserve both balances; create no transaction; mark affected historical calculation incomplete/unavailable until resolved.

**Source IDs:** AC-39; FR-LDG-015; RULE-LDG-005; EC-LDG-026.

### AC-40

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Earlier checkpoint and later opening exist, but intervening activity or ordering is insufficient to calculate the comparison.

**When** the stated action is completed or attempted.

**Then** Show comparison as indeterminate, not a fabricated difference; unknown debt cannot validate an old payment.

**Source IDs:** AC-40; FR-LDG-015; RULE-LDG-005; EC-LDG-027.

### AC-41

**BLOCKED — AUD-LDG-002 / Q-05: historical liability, ordering or settlement boundary.** Baseline expectations below remain review targets.

**Given** Card debt Rp300,000; a Rp200,000 purchase and Rp400,000 payment share the exact timestamp.

**When** the stated action is completed or attempted.

**Then** Ask which happened first. Purchase-first can pass this amount check; payment-first creates an unsupported credit conflict. If order is unknown, show indeterminate and do not commit an operation requiring a definite answer.

**Source IDs:** AC-41; FR-LDG-017; RULE-LDG-006; EC-LDG-028.

### AC-43

**Given** Two equal-time facts yield the same financial result regardless of their applicable order.

**When** the stated action is completed or attempted.

**Then** Do not ask the user solely to order the tie; calculation remains deterministic.

**Source IDs:** AC-43; FR-LDG-017; RULE-LDG-006; EC-LDG-030.

### LEDGER-13.2

**BLOCKED — Q-05 / AUD-LDG-003: settlement coordination.** Baseline expectations below remain review targets.

**Given** a credit purchase of Rp800,000.

**When** recognize purchase then pay Rp800,000 from an asset.

**Then** recognize Rp800,000 expense and liability at purchase with no immediate asset reduction; payment reduces asset and liability and settles obligation without another expense.

**Source IDs:** Ledger §13.2; EC-CC-001; EC-CC-002.

### LEDGER-MATRIX 22.1-1

**BLOCKED — AUD-LDG-001.** Baseline expectations below remain review targets.

**Given** two refunds compete for the last Rp200,000 refundable amount.

**When** attempt both links.

**Then** at most one valid set commits and a failed link creates no second refund inflow.

**Source IDs:** Ledger §22.1-1.

### LEDGER-MATRIX 22.1-3

**BLOCKED — AUD-LDG-002 / AUD-LDG-003.** Baseline expectations below remain review targets.

**Given** a proposed backdated purchase correction invalidates an existing payment.

**When** attempt the correction.

**Then** preserve old purchase, payment, bank balance and settlement until a valid set commits atomically.

**Source IDs:** Ledger §22.1-3.

### LEDGER-MATRIX 27.6-1

**BLOCKED — AUD-LDG-001.** Baseline expectations below remain review targets.

**Given** two concurrent refund links each fit individually but together exceed purchase.

**When** attempt both links.

**Then** at most a valid combination commits with no duplicate inflow, silent link or lost refund.

**Source IDs:** Ledger §27.6-1.

### LEDGER-MATRIX 27.6-3

**BLOCKED — AUD-LDG-002.** Baseline expectations below remain review targets.

**Given** earlier checkpoint Rp700,000 plus complete intervening effects yields Rp950,000 against Rp1,000,000 opening.

**When** compare checkpoint and opening.

**Then** show Rp50,000 discrepancy, incomplete/unavailable history; preserve balances and create no transaction.

**Source IDs:** Ledger §27.6-3.

### LEDGER-MATRIX 27.6-4

**BLOCKED — AUD-LDG-002 / AUD-LDG-004.** Baseline expectations below remain review targets.

**Given** the same comparison has unknown intervening time or order.

**When** compare checkpoint and opening.

**Then** show indeterminate, not a fabricated Rp50,000 discrepancy.

**Source IDs:** Ledger §27.6-4.

### LEDGER-MATRIX 27.6-5

**BLOCKED — AUD-LDG-004.** Baseline expectations below remain review targets.

**Given** an earlier ordering choice becomes inapplicable after an edit.

**When** revalidate history.

**Then** ask again only if ordering affects validity; preserve facts if cancelled or validation fails.

**Source IDs:** Ledger §27.6-5.

### LEDGER-MATRIX 27.6-6

**BLOCKED — AUD-LDG-008.** Baseline expectations below remain review targets.

**Given** a checkpoint correction or archived-account change is committed.

**When** rebuild derived data.

**Then** match committed facts and preserve unavailable or incomplete states.

**Source IDs:** Ledger §27.6-6.
