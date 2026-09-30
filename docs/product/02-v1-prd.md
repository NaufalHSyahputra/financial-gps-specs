# V1 Product Requirements Document

**Document ID:** PRD-V1  
**Status:** APPROVED / FROZEN — Normative

## 1. Objective

Build a single-user PFM V1 that operates as a **Financial GPS** across Past, Now, and Future. The product must combine low-friction transaction capture, cycle budgeting, actual financial state, Safe-to-Spend, and explainable forecasting without mixing actual facts with future assumptions.

## 2. Primary Product Loop

```text
Capture actual transactions
        ↓
Update actual balances + liabilities
        ↓
Update budget consumption + allocations
        ↓
Calculate STS / budget health
        ↓
Project future position from explicit assumptions
        ↓
Surface deterministic contextual insights
        ↓
User adjusts spending/plans/allocations
```

## 3. Product Requirements

### `PRD-TRX-001` Transactions
V1 MUST support Income, Expense, and Transfer semantics at minimum. Transfer between owned accounts is not spending. Credit/BNPL purchase recognizes expense at purchase time; repayment is not a second expense.

### `PRD-TRX-002` Quick expense capture
Default expense flow SHOULD be: `+ → calculator-style amount → Category → Account → Save`. Merchant, notes, tags, custom date/time, attachments are secondary and MUST NOT block ordinary quick entry.

### `PRD-BUD-001` Spending budgets
Users MUST be able to create category/purpose Budgets per Financial Cycle and see Amount, Spent, Remaining/Overspend, utilization, warning state, cycle progress, pace, and deterministic projection when sufficient data exists. Creating a Budget MUST NOT reserve or move money. Total Active Budget MAY exceed current Flexible Money; the system MUST warn but MUST NOT block creation solely for that reason.

### `PRD-BUD-002` Budget reset/template
Budget instances close with their cycle. Unused budget does not create money; unspent actual Flexible remains. New cycle budgets start absent/zero until created. User MAY use previous cycle as a template to create new budget instances.

### `PRD-ALLOC-001` Money allocation
Flexible, Reserved, Locked, and Unallocated represent the role of actual owned money and remain separate from Budget. Newly available money—including starting balance and received income—MUST remain Unallocated until explicitly allocated. Only explicitly Flexible money may participate positively in current STS.

### `PRD-INC-001` Expected income
Expected salary/freelance/other future income is planning data owned by Income Management. It MUST NOT increase current balance, Flexible, or STS. Financial Accounts & Ledger owns the actual Income transaction. When recording Income, the user MAY explicitly link a relevant Expected Income occurrence; the link is optional and unlinked actual Income is valid. Only that explicit claim/link changes the occurrence from `EXPECTED` or `MISSED` to `RECEIVED`. Amount/date/description/source similarity and date arrival MUST NOT auto-link or mark it received in V1.

### `PRD-STS-001` Safe to Spend
STS is deterministic current guidance based on actual Flexible funding and applicable unfunded obligations due within the active Financial Cycle. An obligation due after the cycle boundary does not reduce current STS. Expected Income contributes zero; post-cycle risk is handled by Forecast/Advisor rather than extending the STS horizon.

### `PRD-FCST-001` Forecast
Forecast starts from actual current state and applies future income/outflows/obligations. `EXPECTED_STABLE` income MUST be included in the Base Forecast. `EXPECTED_UNCERTAIN` income MUST be excluded from Base Forecast and included only in explicit Scenario Forecasts. Forecast MUST detect post-cycle Funding Dependency when a future obligation cannot be funded by the actual-only projected path and therefore depends on unreceived Expected Income before the obligation.

### `PRD-CYCLE-001` Financial Cycle
Cycle is configurable by anchor/payday day and weekday adjustment policy. Calendar month is not assumed. Financial Calendar is the sole cycle-boundary authority. A configuration change never retroactively redefines the active cycle. From the boundary immediately following it, a transition under 15 days merges with the following regular cycle as an Extended Cycle; a transition of 15 days or more is a standalone Short Cycle before regular cycles under the new anchor.

### `PRD-OBL-001` Obligations
Support manual, recurring, staged recurring, credit statement, BNPL, and installment obligations. Recurring amount may change by effective period.

### `PRD-ADV-001` Deterministic insights
Contextual insights MUST explain what happened, why it matters, likely deterministic projection, financial impact, and possible corrective action. All numeric advice comes from the Financial Engine.

### `PRD-ONB-001` Short onboarding
Onboarding MUST require only essential cycle/region settings and starting Cash. A default Cash account MAY be created. Other accounts/liabilities are added after onboarding.

## 4. Canonical Example

```text
Actual state
Available Money                   Rp6,800,000
Flexible                          Rp3,800,000
Reserved                          Rp1,000,000
Locked                            Rp2,000,000
Current-cycle obligation          Rp1,550,000
Reservation funding obligation      Rp500,000
Expected salary tomorrow         Rp10,000,000 (forecast only)

Unfunded obligation               Rp1,050,000
Current STS                       Rp2,750,000
```

The salary MUST NOT increase the Rp2,750,000 current STS. Forecast may separately show the projected position after salary receipt.

## 5. Acceptance Outcomes

A developer/AI agent can determine without guessing:
- what counts as actual vs forecast;
- how cycle boundaries adjust;
- how budgets consume actual expenses and reset;
- how STS and Daily STS are calculated;
- how Expected Income affects only forecast until explicitly linked and `RECEIVED`, including Base vs Scenario Forecast;
- how newly available money remains Unallocated until explicit allocation;
- how Budgets may exceed Flexible with a warning without reserving money;
- how Funding Dependency is detected for post-cycle obligations;
- why budget projection is unavailable before 3 elapsed cycle days;
- how credit/BNPL/installment expense and payment differ;
- how recurring staged obligations resolve;
- how deterministic advisor numbers are produced;
- how corrections propagate.

## 6. Non-Goals

See `03-v1-scope.md`. No LLM chatbot, crypto/investment portfolio, bill splitting, advanced accounting, or speculative feature expansion is required.
