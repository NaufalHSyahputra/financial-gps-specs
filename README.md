# Financial GPS — Product & Architecture Specifications

> **Financial GPS, not just an expense tracker.**

This repository is the canonical specification source of truth for **Financial GPS**, a Personal Finance Management (PFM) application designed to help users understand not only **where their money went**, but also:

- where they financially stand today,
- how safely they can spend until the next payday,
- whether their budgets are on track,
- what financial obligations are approaching,
- where their finances are heading, and
- what actions they can take next.

Financial GPS is designed around the progression:

**Past → Now → Future**

Rather than functioning only as an expense history, the product combines actual financial state, budgeting, obligations, forecasting, and deterministic financial guidance.

---

## Repository Purpose

This repository contains the **canonical product, financial engine, UX, domain, and architecture specifications** for Financial GPS.

It is a **specification repository**, not the application source-code repository.

The documents here define the expected behavior of the system before and during implementation, including:

- product behavior,
- business rules,
- financial calculations,
- edge cases,
- domain terminology,
- UX behavior,
- architectural constraints,
- bounded-context ownership,
- deterministic financial rules, and
- Tactical DDD models.

The objective is to make the specification deterministic enough that a software engineer or AI coding agent can implement the product without having to guess core product behavior.

---

## Specification Status

The current documentation baseline is:

**FROZEN — Normative**

A frozen specification represents an approved implementation baseline.

Frozen does **not** mean that the product can never change. It means changes must be intentional, traceable, and reconciled across affected specifications rather than silently introduced during implementation.

The first Tactical DDD bounded-context specification has also been frozen:

**Financial Calendar — Tactical DDD v1.0**

Additional bounded contexts will be specified and frozen incrementally.

---

# Product Thesis

Traditional expense trackers primarily explain the past:

> “Where did my money go?”

Financial GPS extends this into three financial perspectives.

### Past

Understand what already happened.

Examples:

- transactions,
- spending history,
- budget consumption,
- category spending.

### Now

Understand the current financial position.

Examples:

- available money,
- money allocation,
- Safe-to-Spend,
- funding deficits,
- current obligations.

### Future

Understand where finances are heading.

Examples:

- expected income,
- upcoming commitments,
- projected spending,
- financial forecast,
- funding dependency,
- deterministic financial guidance.

The product therefore acts as a **financial navigation system**, rather than only a financial record.

---

# Core Product Principles

## 1. Actual and Forecast Are Strictly Separated

The system distinguishes between:

**Actual financial state**

and

**Projected financial state**

Expected income is not actual money.

Forecast values must never silently mutate actual financial state.

For example:

- an expected salary does not increase current Safe-to-Spend,
- uncertain future income is not treated as available money,
- forecast scenarios do not modify account balances.

---

## 2. Safe-to-Spend Is Based on Actual Money

Safe-to-Spend (STS) represents how much flexible money can safely be used within the current Financial Cycle after considering applicable unfunded obligations.

Conceptually:

```text
RawSTS =
    FlexibleMoney
    - ApplicableUnfundedObligationsWithinCurrentCycle

DisplayedSTS =
    max(0, RawSTS)

FundingDeficit =
    max(0, -RawSTS)
```

Expected income and unallocated money do not increase Safe-to-Spend.

---

## 3. Money Allocation Is Explicit

New liquid money begins as:

```text
UNALLOCATED
```

Money may then be explicitly assigned financial roles such as:

```text
Flexible
Reserved
Locked
```

The system must not silently decide how newly received money should be used.

---

## 4. Budgets and Money Allocation Are Different Concepts

A Budget represents a **spending plan**.

An Allocation represents the **role of actual money**.

Therefore:

```text
Budget ≠ Allocation
```

Creating a budget does not automatically reserve money.

This separation prevents double counting and keeps the financial model explainable.

---

## 5. Financial Guidance Is Deterministic

Financial guidance follows the structure:

```text
WHAT HAPPENED
      ↓
WHY
      ↓
WHAT LIKELY HAPPENS
      ↓
FINANCIAL IMPACT
      ↓
RECOMMENDED ACTION
```

Authoritative financial calculations come from the deterministic financial engine.

AI/LLM functionality may later assist with explanation or structured intent extraction, but it must not become the authoritative financial calculator.

---

# V1 Product Scope

The V1 product includes the core capabilities required to provide the Financial GPS experience.

Major areas include:

- Financial Accounts
- Transactions
- Income
- Expected Income
- Money Allocation
- Budgets
- Financial Commitments
- Credit / PayLater obligations
- Financial Cycles
- Safe-to-Spend
- Forecasting
- Funding Dependency detection
- Deterministic Financial Guidance
- Backup / Export / Import / Restore

V1 deliberately avoids infrastructure or features that are not required to validate the core product.

Examples of intentionally deferred capabilities include:

- automatic multi-device synchronization,
- mandatory backend services,
- user accounts/authentication,
- microservices,
- Event Sourcing,
- distributed messaging infrastructure,
- complex cloud infrastructure.

---

# Architecture Direction

Financial GPS V1 follows a:

**Local-first Modular Monolith**

Primary implementation direction:

```text
Kotlin Multiplatform
        +
Compose Multiplatform
        +
SQLite
        +
SQLDelight
```

Initial first-class platforms:

```text
Android
macOS Desktop
```

The architecture emphasizes:

- deterministic financial calculations,
- explicit domain boundaries,
- rich domain modeling,
- logical CQRS,
- rebuildable derived state,
- synchronous financial correctness,
- testability,
- explainability,
- local-first reliability.

---

# Domain Architecture

Strategic DDD divides the system into explicit Bounded Contexts.

## Core Domain

### Financial Intelligence

Financial Intelligence is the primary Core Domain.

Its responsibilities include:

- Financial Positioning
  - Safe-to-Spend
  - Funding Deficit

- Financial Projection
  - Base Forecast
  - Scenario Forecast

- Risk Detection
  - Funding Dependency
  - Low Safe-to-Spend
  - cross-domain financial risks

- Financial Guidance
  - What Happened
  - Why
  - What Likely Happens
  - Impact
  - Action

Financial Intelligence interprets authoritative facts owned by supporting domains.

It does not autonomously modify those domains.

---

## Supporting Bounded Contexts

### Financial Accounts & Ledger

Owns actual financial facts.

Examples:

- accounts,
- balances,
- actual income,
- expenses,
- transfers,
- payments,
- transaction categories,
- credit facilities,
- current outstanding balances.

---

### Income Management

Owns:

- Income Sources,
- Expected Income,
- income schedules,
- stability classification,
- Expected Income lifecycle.

Expected Income becomes received only through an explicit user link to an actual Income transaction.

Similarity in amount, date, description, or source must not automatically mark Expected Income as received.

---

### Money Allocation

Owns financial roles of actual money:

- Unallocated,
- Flexible,
- Reserved,
- Locked,
- Allocation Shortfall,
- Reserved Coverage.

Money Allocation does not own budgets, forecasts, or Safe-to-Spend.

---

### Spending Planning

Owns:

- Budgets,
- Budget ↔ Category mappings,
- Budget Spent,
- Budget Remaining,
- Overspending,
- Budget Pace,
- Projected Spending,
- Projected Overspending.

---

### Financial Commitments

Owns future financial obligations.

Examples:

- recurring bills,
- credit-card statement obligations,
- PayLater settlements,
- installments,
- repayment schedules,
- mortgage/KPR schedules,
- due dates,
- commitment lifecycle.

---

### Financial Calendar

Financial Calendar is the authoritative temporal domain.

It owns:

- Payday Anchor,
- Payday Adjustment Policy,
- Financial Cycle resolution,
- cycle position,
- configuration transitions,
- Short Cycle,
- Extended Cycle.

Other bounded contexts must not independently reconstruct Financial Cycle semantics.

The Tactical DDD model for this bounded context is frozen as:

```text
Financial Calendar — Tactical DDD v1.0
```

---

# Financial Calendar Tactical DDD

Financial Calendar is the first bounded context to complete Tactical DDD specification.

The frozen specification defines:

- Aggregate Root,
- Entities,
- Value Objects,
- deterministic cycle resolution,
- configuration revision history,
- pending configuration changes,
- historical floor,
- Financial Cycle Position,
- commands,
- queries,
- domain events,
- domain services,
- failure semantics,
- cross-context contracts,
- temporal invariants.

A key design decision is that:

```text
FinancialCycle
```

is a **derived immutable Value Object**, not a persisted Aggregate.

Financial Cycles are deterministically resolved from calendar configuration history.

The authoritative model uses local financial dates:

```text
StartDate
EndDate
```

When an instant/timestamp range is required by another engine or integration layer:

```text
[
    start-of-day(StartDate, FinancialTimezone),
    start-of-day(EndDate + 1 day, FinancialTimezone)
)
```

This preserves a pure calendar-date domain model while providing an explicit integration contract for timestamp-based consumers.

---

# Documentation Authority

Not all documents have equal authority when interpreting product behavior.

The primary precedence is:

```text
Product Requirements Baseline
        ↓
Financial Engine
        ↓
UX / Wireframes
        ↓
Design System
        ↓
UI Screens / Mockups
```

Architecture documents must implement these requirements rather than silently redefine them.

Strategic DDD defines:

- domain ownership,
- bounded-context boundaries,
- domain language,
- context relationships.

Tactical DDD defines implementation-oriented domain models within those approved boundaries.

---

# Frozen Specification Governance

Frozen specifications are normative implementation baselines.

Changes must follow explicit governance.

## Product Behavior Changes

Changes affecting:

- product rules,
- financial behavior,
- formulas,
- lifecycle semantics,
- user-visible financial outcomes,

must go through requirements reconciliation / change control.

They must not be introduced silently through architecture or implementation.

---

## Architecture Changes

Architecture decisions may be documented through:

```text
ADR — Architecture Decision Record
```

An ADR may clarify or change architectural implementation decisions.

However:

> **An ADR cannot override Product Requirements.**

---

## Tactical DDD Clarifications

Tactical modeling may expose ambiguities that were not sufficiently explicit in higher-level requirements.

When that happens:

1. identify the ambiguity,
2. determine the required product behavior,
3. reconcile it with the authoritative requirements,
4. record the approved clarification,
5. then freeze the Tactical DDD decision.

This prevents implementation details from silently becoming product rules.

---

# Immediate Financial Consistency

Financially critical operations require synchronous correctness.

Conceptually:

```text
Command
   ↓
Application Layer
   ↓
Domain Operation
   ↓
Critical Financial Recalculation
   ↓
SQLite Transaction
   ↓
COMMIT
   ↓
Domain Events
   ↓
Secondary Reactions
```

For example:

```text
Record Expense
      ↓
Transaction
      ↓
Account Balance
      ↓
Budget Actual
      ↓
Allocation State
      ↓
Safe-to-Spend
      ↓
COMMIT
```

The application must not report success while financially critical derived state is still inconsistent.

Domain Events are used to communicate facts and trigger reactions.

They are **not** the sole mechanism guaranteeing immediate financial correctness.

---

# Event-Driven, Not Event-Sourced

Financial GPS uses meaningful past-tense Domain Events where appropriate.

However:

```text
Event-Driven ≠ Event Sourcing
```

The application does not reconstruct authoritative financial state exclusively by replaying an event log.

Canonical state remains persisted normally, while derived state must be rebuildable where appropriate.

---

# AI Trust Boundary

Future AI/LLM capabilities must operate behind a strict trust boundary.

Conceptually:

```text
Natural Language
      ↓
LLM Parser
      ↓
Structured Intent / Draft
      ↓
Deterministic Validation
      ↓
User Confirmation
      ↓
Domain Command
      ↓
Authoritative Financial Engine
```

An LLM may propose intent.

It does not receive direct authority to mutate financial state or calculate authoritative financial results.

---

# Recommended Reading Order

For someone new to the project, the recommended order is:

### 1. Product

Start with the Product Vision and Product Requirements Baseline.

Understand:

- why Financial GPS exists,
- the V1 scope,
- core user problems,
- authoritative product behavior.

### 2. Financial Rules

Continue with the Financial Engine specification.

Understand:

- Safe-to-Spend,
- Financial Cycles,
- budgets,
- expected income,
- obligations,
- forecasting,
- calculation rules.

### 3. Domain Model

Read the domain model, glossary, edge cases, and related functional specifications.

Understand the language and invariants used throughout the system.

### 4. UX

Read:

- UX Foundation,
- Information Architecture,
- User Flows,
- Design System.

Understand how domain behavior is exposed to users.

### 5. Architecture Constitution

Understand the engineering constraints that all implementation decisions must respect.

### 6. Strategic DDD

Understand:

- Bounded Contexts,
- domain ownership,
- context relationships,
- Core Domain vs Supporting Domains.

### 7. Tactical DDD

Read each frozen bounded-context Tactical DDD specification.

Currently frozen:

```text
Financial Calendar — Tactical DDD v1.0
```

Additional bounded contexts will be added incrementally.

---

# Tactical DDD Roadmap

The intended Tactical DDD progression is:

```text
1. Financial Calendar              ✓ FROZEN v1.0
2. Financial Accounts & Ledger
3. Income Management
4. Money Allocation
5. Spending Planning
6. Financial Commitments
7. Financial Intelligence
8. Cross-Context Orchestration
9. Command / Event / Query Consistency
10. Global Invariant Audit
```

Each bounded context should be reviewed before being promoted to a frozen specification.

---

# Testing Philosophy

Financial correctness is a first-class architectural concern.

The specifications are designed to support:

- deterministic unit tests,
- aggregate invariant tests,
- state-machine tests,
- architecture fitness tests,
- calculation dependency tests,
- property-based testing where valuable,
- human-readable Golden Financial Scenarios,
- regression testing of financial calculations.

Financial calculations should be reproducible:

> Given the same authoritative facts and configuration, the engine should produce the same financial result.

---

# What This Repository Is Not

This repository is **not**:

- the production application,
- a backend repository,
- a UI implementation repository,
- a collection of brainstorming notes,
- a generic expense-tracker template,
- an Event Sourcing implementation,
- a microservices architecture specification.

It is the **normative specification repository** for Financial GPS.

Implementation repositories should consume these specifications rather than redefine them.

---

# Contributing to Specifications

Before changing a frozen document, determine what kind of change is being proposed.

Ask:

```text
Does this change product behavior?
        ↓
Requirements Change / Reconciliation

Does this change architecture only?
        ↓
ADR

Does this expose an ambiguity?
        ↓
Clarify → Reconcile → Approve → Update Specification

Is this purely editorial?
        ↓
Non-semantic documentation correction
```

Do not silently modify financial behavior while implementing the application.

When multiple documents are affected, reconcile all affected normative specifications to prevent contradictory rules.

---

# Repository Philosophy

The core philosophy of this repository is:

> **Make important financial behavior explicit before implementation.**

A developer should not have to guess:

- what counts as actual money,
- when expected income becomes received,
- how Safe-to-Spend is calculated,
- how obligations affect available money,
- how budgets interact with allocations,
- how Financial Cycles are resolved,
- whether forecast values may affect actual state,
- which bounded context owns a concept.

If implementation requires guessing an important financial rule, the specification is incomplete and should be clarified rather than silently encoded into software.

---

# Current Milestone

The canonical V1 product, financial, UX, design, and architecture baseline is frozen.

Strategic DDD is frozen.

Financial Calendar Tactical DDD is frozen at **v1.0**.

The next Tactical DDD bounded context is:

**Financial Accounts & Ledger**

---

## License

No license has been specified for this repository yet.

Until a license is explicitly added, do not assume that the repository contents are licensed for unrestricted reuse.
