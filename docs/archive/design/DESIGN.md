---
name: Financial GPS Design System
version: 1.0.0
status: Canonical UX Foundation Baseline
owner: Product Design
last_updated: 2026-09-29
---

# Financial GPS Design System

This document is the only canonical design specification for the Expense Budget Tracker interface. It preserves the approved mobile-first UX and visual direction while keeping product behavior and financial calculations in their proper source documents.

## 1. Scope and ownership

This document owns:

- design philosophy and visual language;
- typography, color, spacing, shape, elevation, and iconography;
- layout, responsive behavior, and global navigation presentation;
- component anatomy and visual states;
- interaction, motion, accessibility, and content presentation patterns;
- empty, loading, error, disabled, and success states.

This document does not own:

- business rules or functional requirements;
- financial formulas, thresholds, derived values, or state transitions;
- Safe to Spend, budget, allocation, funding, forecast, or pace calculations;
- data models, API contracts, persistence, or reconciliation logic.

When this document mentions a financial concept, it describes only how an authoritative result is presented. Product Requirements own behavior; the Financial Engine owns calculations.

## 2. Source File Precedence

When sources overlap or conflict, follow this order from highest to lowest authority:

1. **Product Requirements** — product behavior, business rules, user flows, functional requirements, and edge cases.
2. **Financial Engine** — formulas, derived values, calculation rules, financial states, forecasts, and transitions.
3. **UX and wireframes** — information architecture, screen hierarchy, navigation, interaction flow, and user journeys.
4. **This canonical `design/DESIGN.md`** — visual language, components, layout, typography, spacing, color, motion, and accessibility.
5. **UI screens and mockups** — examples of the approved system in use.

If a lower-priority artifact conflicts with a higher-priority source, update the lower-priority artifact. Never alter a higher-priority rule merely to match a mockup. If Product Requirements or Financial Engine documentation is unavailable, do not infer or invent its rules from this document; flag the dependency for resolution.

## 3. Single Source of Truth Principle

Every concept has exactly one canonical owner:

- Product Requirements own what the product does.
- The Financial Engine owns how financial results are calculated.
- UX specifications own navigation, screen hierarchy, and flows.
- This file owns the design system and presentation rules.
- Mockups demonstrate the rules but do not redefine them.

There must be exactly one file named `DESIGN.md` in the repository: this file. Feature folders may contain implementation notes, but they must link here rather than copy tokens or component rules.

## 4. Avoid Documentation Drift

- Change a rule only in its canonical owner.
- Link to canonical rules; do not duplicate them in feature documentation.
- Do not copy formulas, financial thresholds, or business logic into design documentation.
- When a token or component contract changes, update this file and all affected mockups in the same change.
- When product behavior changes, update Product Requirements first; update UX, this file, and mockups only as downstream work.
- Resolve conflicts by precedence and record material changes in `CHANGELOG.md`.
- Reviews must verify that no additional `DESIGN.md` exists and no lower-priority artifact has silently become normative.

## 5. Design philosophy

The **Financial GPS Design System** is calm, minimalist, spacious, card-based, and highly structured. It helps people understand their financial position and choose an action; it does not act autonomously on their behalf.

### Core principles

1. **Explain before action.** Important guidance follows: what happened, why it matters, what the user can do, and clear actions.
2. **Actual and forecast remain distinct.** Actual money and posted activity must never be visually confused with expected income, assumptions, or scenarios.
3. **Safe to Spend is the daily hero.** It receives the strongest emphasis on Home; supporting allocation and guidance remain subordinate.
4. **User-controlled money movement.** Allocating, reserving, or transferring money always requires an explicit review and confirmation.
5. **Exception-driven guidance.** Calm confirmation is concise; warnings become prominent only when attention or a decision is required.
6. **Progressive disclosure.** Show the useful answer first, with explanations and detail available on demand.
7. **Cognitive quiet.** Prefer whitespace, restrained neutrals, short copy, and a small number of meaningful accents over decorative dashboards or gamification.

## 6. Foundations

### 6.1 Typography

Use **Plus Jakarta Sans** for all interface text and numeric content. Fallback stack: `system-ui, -apple-system, BlinkMacSystemFont, sans-serif`. Do not mix Inter or a second UI typeface into the product.

| Token | Size / line height | Weight | Use |
|---|---:|---:|---|
| `display-hero` | 32 / 36 px | 800 | Safe to Spend and primary financial hero values |
| `display-metric` | 28 / 32 px | 700 | Major forecast or summary values |
| `page-title` | 26 / 32 px | 700 | Primary screen title |
| `heading-lg` | 20 / 26 px | 700 | Major section heading |
| `heading-md` | 18 / 24 px | 700 | Card and section title |
| `title-sm` | 16 / 22 px | 600 | Component title or important metric label |
| `body` | 14 / 20 px | 400 | Default content and explanations |
| `body-medium` | 14 / 20 px | 500 | Emphasized body and row labels |
| `label` | 12 / 16 px | 600 | Controls, compact statuses, and metadata |
| `caption` | 11 / 14 px | 500 | Timestamps and supporting notes |

Hero values may use 30 px at narrow widths below 360 px. Use tight negative tracking only for display and page-title tokens. Avoid all-caps prose; uppercase is reserved for short status labels.

All money, percentages, dates with changing digits, and side-by-side metrics use tabular figures: `font-variant-numeric: tabular-nums`. Follow the active locale for currency and separators. Prefix inflows and outflows only when direction would otherwise be ambiguous. Expected values always include a visible `PLANNED`, `EXPECTED`, or `SCENARIO` label.

### 6.2 Color tokens

#### Neutral and action palette

| Token | Value | Use |
|---|---|---|
| `surface-canvas` | `#F8F9FA` | App background |
| `surface-card` | `#FFFFFF` | Cards, sheets, and primary content |
| `surface-subtle` | `#F1F3F5` | Grouped rows and secondary controls |
| `surface-muted` | `#E9ECEF` | Disabled tracks and subtle dividers |
| `border-subtle` | `#E5E7EB` | Default card and control border |
| `border-strong` | `#CBD5E1` | Focused or structurally important border |
| `text-primary` | `#111827` | Headings, values, and primary text |
| `text-secondary` | `#6B7280` | Explanations and supporting labels |
| `text-muted` | `#94A3B8` | Placeholders and tertiary metadata |
| `action-primary` | `#1E232A` | Primary button and selected control |
| `on-action-primary` | `#FFFFFF` | Content on primary actions |

#### Semantic palette

| Token | Value | Meaning |
|---|---|---|
| `semantic-positive` | `#10B981` | Actual received income, healthy state, on-track result |
| `semantic-planned` | `#0284C7` | Stable expected income and future assumptions |
| `semantic-caution` | `#D97706` | Pace concern, funding dependency, due boundary |
| `semantic-critical` | `#E11D48` | Deficit, overspend, missed income, shortfall |
| `semantic-info` | `#64748B` | Neutral information and uncertain scenario |
| `role-flexible` | `#1E293B` | Flexible allocation |
| `role-reserved` | `#7C3AED` | Protected or reserved allocation |
| `role-locked` | `#334155` | Restricted or locked allocation |
| `role-unallocated` | `#CBD5E1` | Money without assigned intent |

Semantic rules:

- Green represents actual received income or healthy states, never expected income.
- Blue represents planned or expected values and is paired with explicit text.
- Amber indicates attention or dependency, not a confirmed deficit.
- Red is reserved for critical, realized, or high-confidence negative states.
- Color never communicates state alone; pair it with a label and, where helpful, an icon.
- Use pale semantic containers with dark text for warnings; avoid large saturated fields.

Dark mode is not part of this baseline. Do not infer a dark palette by inverting these tokens.

### 6.3 Spacing and grid

Use a 4 px base unit:

| Token | Value |
|---|---:|
| `space-1` | 4 px |
| `space-2` | 8 px |
| `space-3` | 12 px |
| `space-4` | 16 px |
| `space-5` | 20 px |
| `space-6` | 24 px |
| `space-8` | 32 px |

- Standard page padding: 16 px; Planning may use 20 px for its workspace sections.
- Narrow widths below 360 px: 12 px page padding.
- Card padding: 16 px; compact advisory and list cards: 12–14 px.
- Card stack gap: 12–16 px; related content inside a card: 8–12 px.
- Maintain at least 24 px between major page sections.
- Reserve bottom padding for the navigation and device safe area.

### 6.4 Shape, borders, and elevation

| Token | Value | Use |
|---|---:|---|
| `radius-sm` | 4 px | Compact badges |
| `radius-md` | 8 px | Buttons and dense controls |
| `radius-lg` | 12 px | Rows, inputs, and advisory cards |
| `radius-xl` | 16 px | Primary cards and sheets |
| `radius-full` | 9999 px | Pills, avatars, and progress tracks |

Cards use a 1 px subtle border. Default elevation is flat or a restrained shadow no stronger than `0 2px 8px rgba(15, 23, 42, 0.05)`. Overlays and bottom sheets may use `0 8px 24px rgba(15, 23, 42, 0.08)`. Avoid heavy shadows, glossy effects, or skeuomorphism.

### 6.5 Iconography

Use one consistent outline icon family with rounded line caps and 1.75–2 px strokes. Common icons are 20 px; compact controls may use 16 px. Icons supplement labels and do not replace essential text. Decorative emoji are not part of the canonical visual system; status symbols in copy must have accessible labels.

## 7. Layout and responsive behavior

- Design mobile-first for 390–393 px reference viewports and support 320–428 px without horizontal scrolling.
- On larger widths, center the content column and cap primary app content at 428 px unless a future tablet specification explicitly introduces a wider layout.
- Respect top and bottom safe-area insets.
- Keep the current financial status and hero content in the upper optical field.
- Content must scroll independently of persistent navigation.
- Do not bake device frames, status bars, or fixed 844/852 px heights into production layouts; those belong only to mockup presentation.
- Text may wrap and cards may grow. Never clip financial values or explanatory copy to preserve a fixed mockup height.

## 8. Global navigation

The app has five primary destinations plus a centered quick-action control:

`Home · Transactions · (+) Add · Budget · Planning · Accounts`

- `Home`, `Transactions`, `Budget`, `Planning`, and `Accounts` are destinations.
- `Add` opens the calculator-first transaction flow and is not a tab destination.
- Active destinations use primary text and a semibold label; inactive destinations use secondary text.
- The Add control remains visually distinct but must not overpower the Safe to Spend hero.
- Navigation persists on primary screens, respects safe-area padding, and exposes a clear accessible name for every item.

## 9. Component contracts

### 9.1 Safe to Spend hero card

The Home hero contains a label, the authoritative Safe to Spend result, optional daily guidance, concise allocation context, and a `Why this amount?` disclosure. The hero value uses `display-hero`, tabular figures, and the strongest contrast on the page.

The explanation view presents the named inputs supplied by Product Requirements and the Financial Engine. It may show a readable breakdown but must not define, simplify, or independently calculate the formula. Expected income is visually identified as not actual according to the authoritative product rules.

### 9.2 Financial GPS guidance card

On-track guidance is a quiet one-line confirmation. Warnings and decision states use this content order:

1. what happened;
2. why it matters;
3. what the user can do;
4. one primary action and, only when useful, one secondary action.

Use neutral, factual language. Do not expose internal confidence scores, data-integrity stamps, actuarial terminology, or invented precision.

### 9.3 Budget health card

Show the budget name, actual usage, budget total, progress visualization, and the pace/status result provided by the Financial Engine. Pair status color with explicit text. Early-cycle projection availability follows Product Requirements; the UI uses a neutral unavailable/loading treatment and does not invent a projection.

### 9.4 Planning workspace

Planning answers “Where is my financial future heading?” and remains visually distinct from transaction history. Its hierarchy is:

1. forecast summary with clear assumption context;
2. funding dependency or boundary guidance when present;
3. expected income, visibly marked as future assumptions;
4. scenarios where supported;
5. upcoming chronological obligations as supporting detail.

Use `Forecast`, `Timeline`, and `Scenarios` as progressive views when needed. A funding dependency presents an explanation before actions; `View Forecast` reveals context and `Reserve Money` opens the allocation flow. It never moves money immediately.

### 9.5 Allocation flow

Show the source, destination role, editable amount, and the remaining unallocated amount before confirmation. Quick percentage controls may accelerate entry but never confirm automatically. The final button includes the action and amount. Success returns a concise confirmation; failures preserve the entered values.

### 9.6 Transaction entry

Use a calculator-first amount entry followed by transaction type, category, account, date, and optional note as defined by Product Requirements. Do not introduce budget assignment unless the canonical product flow explicitly requires it. Keep the primary save action reachable above the safe area and prevent duplicate submission while saving.

### 9.7 Cards and rows

- Primary cards: white surface, 16 px radius, subtle border, 16 px padding.
- Compact advisory cards: pale semantic surface, 12 px radius, 12–14 px padding.
- Rows: at least 44 px high, with values right-aligned and labels left-aligned.
- Dividers are optional and subtle; spacing should establish grouping before borders.
- Entire rows may be tappable only when their destination or action is clear.

### 9.8 Buttons and controls

- Primary button: dark fill, white text, minimum 48 px height, 12 px radius.
- Secondary button: white or subtle fill, 1 px border, primary text, minimum 44 px height.
- Tertiary action: text treatment with a 44 px minimum hit area.
- Destructive actions require explicit red treatment and confirmation proportional to impact.
- Inputs use a white or subtle surface, 1 px border, 12 px radius, visible label, helper/error space, and a clear focus ring.
- Disabled controls remain legible and expose disabled state programmatically.

### 9.9 Badges, chips, and progress indicators

Badges are short, explicit, and non-interactive. Filter chips and segmented controls are interactive, expose selected state, and use a minimum 44 px hit area even when visually compact. Progress bars always have a text equivalent and must not imply a financial threshold not supplied by the authoritative source.

### 9.10 Sheets, dialogs, and explanations

Use bottom sheets for mobile contextual detail and short tasks; use full-screen flows for complex entry. Sheets include a title, clear dismissal, safe-area padding, and focus management. Dialogs are reserved for short confirmations or blocking decisions. Never rely on swipe-only dismissal.

## 10. Interaction patterns

- Provide immediate visual feedback for press, selection, and input.
- Preserve user input across validation and recoverable errors.
- Require explicit confirmation for money movement and destructive actions.
- Avoid surprise navigation after selecting a chip or filter.
- Use inline disclosure for “why” explanations before sending users to another screen.
- Keep one clear primary action per surface; two actions are acceptable when they represent “understand” and “act.”
- Do not use dark patterns, urgency without an authoritative state, or celebratory gamification for routine financial behavior.

## 11. Motion

Motion supports orientation and feedback, never decoration:

- micro feedback: 100–150 ms;
- control and card transitions: 150–200 ms;
- sheets and page-level transitions: 200–300 ms;
- standard easing: `cubic-bezier(0.2, 0, 0, 1)`.

Do not animate changing money values as a slot machine or count-up. Avoid parallax, bounce, and continuous pulsing. Under `prefers-reduced-motion: reduce`, remove nonessential transforms and use immediate or cross-fade state changes.

## 12. System states

### 12.1 Empty states

Empty states are calm, specific, and action-oriented. They include a short heading, one sentence of context, and at most one primary action.

- No transactions: explain that nothing has been recorded for the current scope and offer Add Transaction.
- No budgets: explain the benefit of creating a cycle budget and offer Create Budget.
- No upcoming obligations: confirm that none are currently recorded; no forced CTA is required.
- No scenarios: explain what a scenario helps compare and offer Create Scenario only if supported.
- No guidance issues: show a compact on-track confirmation rather than a large illustration.

Do not portray zero data as an error. Do not use decorative illustrations that dominate the financial context.

### 12.2 Loading states

- Prefer skeletons that match final card and row geometry for initial loads.
- Use inline progress for local actions such as save or refresh.
- Keep existing content visible during background refresh when safe.
- Disable repeat submission and change the action label to a clear progress state.
- Do not display fake financial values in skeletons.
- Announce meaningful loading changes to assistive technology without repeatedly interrupting the user.

### 12.3 Error states

- State what could not be completed, preserve known-good content, and offer a relevant retry or recovery action.
- Field errors appear beside the field and are summarized when submission fails.
- Financial data that may be stale is labeled with its last successful refresh context if that information is available from the product.
- Never replace an unavailable authoritative result with a guessed value.
- Critical errors use red only when the condition is genuinely critical; connectivity and temporary service issues normally use neutral or caution treatment.

### 12.4 Success and confirmation

Confirm completed mutations concisely and show the result. Use a transient message for low-risk changes and a persistent summary for money movement. Do not use confetti or exaggerated reward patterns.

## 13. Accessibility

The baseline target is WCAG 2.2 AA.

- Normal text contrast: at least 4.5:1; large text and meaningful UI graphics: at least 3:1.
- Touch targets: at least 44 × 44 px with adequate separation.
- Support text resizing to 200% without loss of content or function.
- Maintain logical heading structure, reading order, labels, and landmarks.
- Provide visible focus indicators and full keyboard operation for web implementations.
- Use native controls where possible and expose name, role, value, selected, expanded, invalid, and disabled states.
- Never encode actual versus expected, positive versus critical, or allocation roles by color alone.
- Charts and progress indicators require concise textual equivalents.
- Error messages identify the field and correction; do not rely on color or icons alone.
- Dynamic confirmations and errors use appropriate live-region behavior without excessive announcements.
- Respect reduced motion and platform contrast preferences where supported.

## 14. Content style

Use plain, direct language. Explain the situation before asking for action. Prefer “Expected salary has not been received yet” over internal terms such as “funding dependency state transition.” Labels use sentence case except compact status badges. Avoid blame, alarmism, opaque confidence language, and claims that the system can guarantee future outcomes.

## 15. Implementation and review checklist

Before accepting a screen or component:

- confirm behavior against Product Requirements and calculations against the Financial Engine;
- confirm navigation and hierarchy against UX specifications;
- use only canonical tokens and Plus Jakarta Sans;
- distinguish actual, expected, and scenario values in text as well as color;
- verify loading, empty, error, disabled, and success states;
- test at 320, 390, and 428 px widths and with 200% text sizing;
- test keyboard/focus behavior where applicable and screen-reader names for controls;
- verify reduced motion and sufficient contrast;
- confirm money movement requires explicit review and confirmation;
- confirm no duplicate design rules or additional `DESIGN.md` files were introduced.
