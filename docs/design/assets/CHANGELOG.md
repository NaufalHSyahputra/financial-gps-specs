# Design System Consolidation Changelog

## 2026-09-29 — Canonical design source of truth

### Summary

Consolidated the two competing design documents into `design/DESIGN.md`. The approved screens and visual direction were preserved. No HTML or PNG screen artifact was redesigned or modified.

### Repository structure

- Added `design/DESIGN.md` as the sole canonical design source.
- Removed `autonomous_financial_gps/DESIGN.md` and its now-empty folder.
- Removed `financial_precision_engine/DESIGN.md` and its now-empty folder.
- Added this root `CHANGELOG.md` so future design-system changes have a durable audit trail.

**Rationale:** Multiple files named `DESIGN.md` created unclear ownership and allowed typography, color, component, and financial rules to conflict.

### Naming and philosophy

- Renamed **Autonomous Financial GPS** to **Financial GPS Design System**.
- Removed “autonomous” positioning and made user-controlled financial action explicit.
- Preserved the approved calm, minimalist, spacious, card-based, mobile-first visual direction.
- Retained Safe to Spend as the Home hero, exception-driven guidance, actual-versus-forecast separation, and explain-before-action behavior.

**Rationale:** The product explains, forecasts, and guides; it does not take financial decisions or move money for the user.

### Source ownership and governance

- Added **Source File Precedence** in this order: Product Requirements, Financial Engine, UX/wireframes, canonical Design System, UI screens/mockups.
- Added **Single Source of Truth Principle** with one canonical owner per concept.
- Added **Avoid Documentation Drift** rules for linking instead of copying, downstream updates, conflict resolution, and review checks.
- Added an explicit rule to stop and flag missing Product Requirements or Financial Engine documentation rather than infer business logic from visual artifacts.

**Rationale:** These rules prevent lower-priority mockups or design guidance from silently redefining product behavior or calculations.

### Financial and business logic removal

- Removed embedded formulas and arithmetic ownership from the design documentation.
- Removed rules that attempted to calculate Safe to Spend, budget pace, forecasts, funding states, and allocation outcomes inside the design system.
- Removed the prior “calculation waterfall” formula and replaced it with a presentation contract that consumes authoritative named inputs.
- Removed financial threshold definitions such as pace differences and hard-coded status computation.
- Removed confidence stamps, data-integrity scores, “mathematical exactness” claims, actuarial language, and autonomous routing concepts.
- Kept only presentation guidance for authoritative results and references to Product Requirements and the Financial Engine as owners.

**Rationale:** A design system must describe how results appear, not compete with the calculation or product specifications.

### Typography

- Standardized all interface typography on **Plus Jakarta Sans** with system fallbacks.
- Removed the conflicting Plus Jakarta Sans plus Inter split.
- Consolidated the type scale around the approved screens: 32 px hero, 28 px major metric, 26 px page title, 18–20 px headings, 14 px body, and 11–12 px labels/captions.
- Preserved tabular figures for financial values and clarified localized currency presentation.

**Rationale:** One family matches the approved mockups and eliminates inconsistent data-versus-interface typography.

### Color

- Consolidated neutral surfaces around the approved `#F8F9FA`, white cards, gray borders, and near-black text/action palette.
- Unified semantic meanings: emerald for actual received income/healthy state, sky for planned/expected, amber for caution/dependency, rose for critical, and slate for neutral information.
- Unified allocation roles: flexible, reserved, locked, and unallocated.
- Removed conflicting mappings in which blue, green, and slate represented different concepts across the two source files.
- Explicitly excluded dark mode from the baseline rather than retaining an incomplete “night telemetry” palette.

**Rationale:** Each color now has one stable meaning and matches the restrained approved visual direction.

### Spacing, shape, and elevation

- Standardized spacing on a 4 px scale from 4 to 32 px.
- Standardized page padding, card padding, section gaps, and narrow-screen behavior.
- Unified radii at 4, 8, 12, 16, and full-pill values.
- Preserved subtle borders and low-contrast shadows; removed conflicting “micro-radii” and stronger instrumentation styling.

**Rationale:** The resulting system reflects the actual screen artifacts and provides implementation-ready tokens without duplicate scales.

### Navigation and responsive layout

- Clarified the navigation model as five destinations plus one centered Add action.
- Preserved Home, Transactions, Budget, Planning, and Accounts as first-class destinations.
- Clarified that Add launches a calculator-first flow and is not a sixth destination tab.
- Added support expectations for 320–428 px widths, safe areas, content scrolling, and text growth.
- Distinguished mockup-only device frames and fixed heights from production layout rules.

**Rationale:** The prior document called the navigation “five-destination” while listing six controls and risked copying presentation-frame constraints into implementation.

### Components and interaction patterns

- Consolidated contracts for the Safe to Spend hero, explanations, guidance cards, budget health, Planning workspace, allocation, transaction entry, cards, rows, buttons, controls, badges, progress, sheets, and dialogs.
- Preserved the approved Planning hierarchy: forecast, dependency guidance, expected income, scenarios, then upcoming obligations.
- Preserved dual Funding Dependency actions—understand through forecast and act through the allocation flow—without automatic reservation.
- Preserved calculator-first transaction entry and excluded budget assignment unless Product Requirements explicitly require it.
- Added input preservation, duplicate-submit prevention, destructive confirmation, and one-primary-action guidance.

**Rationale:** The contracts retain approved UX while separating visual behavior from financial computation.

### Empty, loading, error, and success states

- Expanded empty-state guidance for transactions, budgets, obligations, scenarios, and on-track guidance.
- Added skeleton, background refresh, inline progress, and assistive announcement rules for loading.
- Added recoverable error, stale-data, validation, retry, and no-guessed-value rules.
- Added restrained success and money-movement confirmation behavior.

**Rationale:** The source documents did not consistently define the full state lifecycle needed for implementation.

### Accessibility and motion

- Consolidated WCAG 2.2 AA requirements for contrast, 44 px targets, 200% text resizing, focus, keyboard support, semantic controls, live regions, and text alternatives.
- Required status communication beyond color and textual equivalents for charts/progress.
- Added reduced-motion behavior and restrained transition timing.
- Prohibited slot-machine money animation, decorative continuous motion, and swipe-only dismissal.

**Rationale:** Accessibility is now a testable component contract rather than a short general statement.

### Files intentionally unchanged

- All `code.html` screen implementations remain unchanged.
- All `screen.png` mockups remain unchanged.
- Both `planning/` and `planning_forward_financial_workspace/` remain in the archive because consolidating design documentation did not authorize deleting screen artifacts; the forward workspace remains the approved directional reference.

**Rationale:** The requested work was documentation consolidation and repository source-of-truth cleanup, not a visual redesign or screen deletion pass.
