# V1 Product Analytics Specification

**Document ID:** AN-V1  
**Status:** APPROVED / FROZEN — Normative

## 1. Objective
Measure whether V1 functions as a Financial GPS: users can capture reality, budget, understand current STS, plan future cash flow, and act on deterministic insights.

## 2. Privacy Rules
Product analytics MUST NOT send exact balances, STS, transaction/income/obligation amounts, account names, merchant/free-text notes, card/account numbers, credentials, or precise location by default. Analytics failure never blocks financial functionality.

## 3. Core Metrics
- `MET-ONB-001` onboarding completion.
- `MET-TRX-001` quick-entry completion and median interaction/time-to-save where instrumentable without financial payload.
- `MET-BUD-001` cycle budget adoption.
- `MET-BUD-002` budget-health view engagement.
- `MET-GPS-001` valid STS view rate.
- `MET-GPS-002` STS explanation engagement.
- `MET-FCST-001` Planning/Forecast adoption.
- `MET-INS-001` contextual insight view/action rate.
- `MET-QUAL-001` authoritative calculation failure rate.

## 4. Events

### `onboarding_completed`
Properties: cycle adjustment policy, default cash created boolean; no balance.

### `transaction_created`
Properties: `transaction_type`, `quick_entry` boolean, optional generic category presence boolean; no amount/description.

### `budget_created`
Properties: `source manual|previous_cycle_template`, category_count bucket.

### `budget_status_viewed`
Properties: `limit_status ON_TRACK_LIMIT|APPROACHING_LIMIT|LIMIT_REACHED|OVERSPENT|OVERSPENT_UNBUDGETED`, `pace_status ON_PACE|AHEAD_OF_PLAN`, `projection_status NOT_YET_ELIGIBLE|AVAILABLE|INSUFFICIENT_DATA`, `budget_exceeds_flexible` boolean. No exact monetary difference.

### `budget_template_applied`
Properties: budget_count bucket; no amounts.

### `allocation_updated`
Properties: allocation type, operation; no amount.

### `allocation_shortfall_detected|resolved`
Properties: cause/action type; no amount.

### `expected_income_created`
Properties: `certainty EXPECTED_STABLE|EXPECTED_UNCERTAIN`, recurring boolean, `forecast_role BASE|SCENARIO_ONLY`.

### `expected_income_linked_received`
Emitted only after the user explicitly links actual Income to an Expected Income occurrence and it becomes `RECEIVED`. Properties: prior state `EXPECTED|MISSED`, certainty, timing variance bucket, amount variance direction `lower|equal|higher`; no exact values. Similarity detection or date arrival MUST NOT emit this event.

### `sts_viewed`
Properties: `sts_state positive|zero|deficit|unresolved`, has_current_cycle_obligation, has_reservation, has_expected_income_in_forecast. Exact STS prohibited.

### `sts_explanation_viewed`
Same categorical state properties; may include `expected_income_exclusion_shown=true`.

### `forecast_viewed`
Properties: contains_stable_expected_income, contains_uncertain_income_scenario, contains_projected_deficit, contains_recurring_obligation.

### `insight_viewed`
Properties: insight_type (`budget_limit|budget_pace|funding_deficit|allocation_shortfall|forecast_pressure|other`), numeric_action_available boolean.

### `funding_dependency_detected`
Properties: `obligation_source_type`, `supporting_income_certainty EXPECTED_STABLE`, `base_forecast_still_deficit` boolean. No amounts.

### `funding_dependency_resolved`
Properties: `resolution_type actual_funding|obligation_changed|income_missed_or_moved|obligation_settled_or_cancelled|other`. No amounts.

### `insight_action_started`
Properties: insight_type, action_type. Event does not imply financial health improvement.

## 5. Interpretation Guardrails
Analytics may show feature usage and state transitions. It cannot by itself prove financial-health improvement or justify borrowing/unlocking savings. Experiments MUST NOT silently change financial formulas, cycle rules, budget thresholds, or STS semantics without a versioned financial-rule review.
