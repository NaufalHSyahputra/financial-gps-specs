# Strategic DDD v1.0

**Status:** APPROVED / FROZEN\
**Version:** 1.0\
**Project:** Personal Finance Management / Expense Budget Tracker\
**Product Thesis:** *Financial GPS, not just an expense tracker.*\
**Authority Level:** Canonical Strategic Domain Model\
**Next State:** Tactical DDD

------------------------------------------------------------------------

## 1. Purpose

This document defines the approved **Strategic Domain-Driven Design
(DDD)** model for V1.

Its purpose is to establish:

-   the Core Domain and Supporting Subdomains;
-   approved Bounded Contexts;
-   authoritative ownership of major business concepts;
-   relationships and dependency direction between contexts;
-   cross-context integration principles;
-   boundaries between financial facts, interpretation, and
    user-directed actions.

This document intentionally does **not** define aggregates, entities,
value-object structures, database schemas, package structures, concrete
event catalogs, APIs, or persistence models. Those belong to later
Tactical DDD and technical-design stages.

------------------------------------------------------------------------

## 2. Relationship to Existing Baselines

Strategic DDD MUST implement the already locked product and architecture
decisions rather than redefine them.

When interpreting this document:

1.  Product Requirements remain authoritative for business behavior,
    user-visible rules, flows, and edge cases.
2.  Financial Engine requirements remain authoritative for financial
    formulas, calculation semantics, derived values, and state
    transitions.
3.  UX Foundation remains authoritative for information architecture,
    hierarchy, navigation, and interaction intent.
4.  Design System remains authoritative for visual and interaction
    presentation.
5.  Architecture Constitution v1.0 remains authoritative for
    architectural constraints and engineering principles.
6.  This Strategic DDD document defines **domain ownership, language
    boundaries, and context relationships** within those constraints.

An architectural/domain model MUST NOT silently change a
higher-authority product rule.

------------------------------------------------------------------------

## 3. Strategic Domain Thesis

The application is not primarily an expense tracker, budget tracker, or
forecasting tool.

Its Core Domain exists to transform financial facts and plans into an
understandable financial position, future trajectory, detected risks,
and actionable guidance.

The intended reasoning loop is:

``` text
What happened?
      ↓
Where did the money go?
      ↓
Am I following my plan?
      ↓
What is my financial position now?
      ↓
If this continues, what is likely to happen?
      ↓
Is there a financial problem or dependency?
      ↓
What should I pay attention to or do?
```

This complete loop is what makes the product a **Financial GPS**.

Guidance is therefore not an optional presentation layer on top of
forecasting. It is an essential output of the Core Domain.

------------------------------------------------------------------------

## 4. Domain Classification

### 4.1 Core Domain --- Financial Intelligence

**Financial Intelligence** is the Core Domain.

It answers:

> Given everything currently known about the user's finances, what is
> their financial position, where are they heading, what risks or
> dependencies exist, and what should they pay attention to?

Its major internal capabilities are:

``` text
Financial Intelligence
│
├── Financial Positioning
│   ├── Safe-to-Spend
│   └── Funding Deficit
│
├── Financial Projection
│   ├── Base Forecast
│   └── Scenario Forecast
│
├── Risk Detection
│   ├── Funding Dependency
│   ├── Low Safe-to-Spend
│   └── Cross-domain financial risks
│
└── Financial Guidance
    ├── What Happened
    ├── Why
    ├── Likely Happens
    ├── Impact
    └── Action
```

For V1, these capabilities are defined as **internal modules of one
Financial Intelligence Bounded Context**, rather than separate Bounded
Contexts.

This keeps the Financial GPS reasoning pipeline cohesive while avoiding
premature context fragmentation.

### 4.2 Supporting Subdomains

The Supporting Subdomains are:

1.  **Financial Accounts & Ledger** --- what financial accounts exist
    and what actually happened financially?
2.  **Income Management** --- what income does the user receive or
    expect?
3.  **Money Allocation** --- what role does actual money have?
4.  **Spending Planning** --- is spending proceeding according to plan?
5.  **Financial Commitments** --- what future payments is the user
    committed to?
6.  **Financial Calendar** --- what financial period does a date belong
    to?

These domains are necessary to support Financial Intelligence but are
not themselves the primary product differentiation.

### 4.3 Technical Capabilities Are Not Business Subdomains

Local persistence, SQLDelight, SQLite, backup/restore, notifications,
secure storage, and other platform/infrastructure concerns MUST NOT be
represented as business subdomains merely to complete a domain diagram.

They remain application/platform/infrastructure capabilities unless
later product requirements establish independent business meaning for
them.

------------------------------------------------------------------------

## 5. Bounded Contexts

V1 defines seven Bounded Contexts.

### 5.1 Financial Accounts & Ledger

**Primary question:**\
\> What financial accounts exist, and what actually happened
financially?

Conceptual ownership includes:

-   financial accounts;
-   asset accounts such as Cash, Bank, and E-Wallet;
-   revolving credit facilities represented as accounts, including
    Credit Card and PayLater;
-   actual Income, Expense, Transfer, and Payment transactions;
-   transaction categories;
-   current account state;
-   current credit outstanding.

This context is authoritative for **actual monetary facts**.

An expected income is not actual money until represented by an actual
Ledger fact.

A credit-card or PayLater account may contain specialized financial
terms such as credit limit, statement/settlement timing, and payment-due
information. The exact Tactical DDD representation is intentionally
deferred.

#### Credit Account Semantics

Credit Card and PayLater are treated as financial accounts/facilities
with liability semantics rather than asset semantics.

A credit purchase creates an actual expense and increases
liability/outstanding, but does not reduce cash at purchase time.

Repayment reduces cash and liability but MUST NOT create the same
expense a second time.

------------------------------------------------------------------------

### 5.2 Income Management

**Primary question:**  
> What income does the user receive or expect?

Conceptual ownership includes:

- income sources;
- expected income;
- recurring income expectations;
- expected-income schedules;
- income stability classification;
- expected-income lifecycle such as expected, received, and missed.

Income Management owns the **expectation and lifecycle**, while Financial Accounts & Ledger owns the actual income transaction.

Therefore:

```text
Expected Income ≠ Actual Money
```

Expected income MAY affect forecast calculations according to Financial Engine rules, but MUST NOT become actual financial position merely because its expected date arrives.

#### Expected Income Realization

V1 uses an **Explicit Claim / Manual Link on Record** mechanism.

When the user records an actual `Income` transaction in Financial Accounts & Ledger, the user MAY explicitly associate that transaction with a relevant Expected Income occurrence.

```text
Expected Income (EXPECTED)
        │
        │ user records actual Income
        │ and explicitly links it
        ▼
Actual Income Transaction
        │
        ▼
Expected Income (RECEIVED)
```

The actual Income transaction remains owned by Financial Accounts & Ledger. The Expected Income lifecycle remains owned by Income Management.

The link is optional. An actual Income transaction MAY exist without realizing any Expected Income.

V1 MUST NOT automatically mark an Expected Income as `RECEIVED` based on amount similarity, date proximity, description, source name, or another fuzzy/heuristic matching mechanism.

Automatic fuzzy matching is explicitly out of scope for V1.

### 5.3 Money Allocation

**Primary question:**\
\> What role does the actual money the user has play?

Conceptual ownership includes:

-   Unallocated;
-   Flexible;
-   Reserved;
-   Locked;
-   allocation state;
-   allocation shortfall.

Money Allocation does **not** own:

-   budgets;
-   Safe-to-Spend;
-   forecasts;
-   financial guidance.

Creating or changing a Budget MUST NOT implicitly reserve or allocate
money.

Money Allocation provides authoritative allocation state to Financial
Intelligence, but it does not decide whether the user is financially
safe.

------------------------------------------------------------------------

### 5.4 Spending Planning

**Primary question:**\
\> Is the user spending according to plan?

Conceptual ownership includes:

-   Budget;
-   budget amount;
-   Budget-to-Category mapping;
-   Budget Spent;
-   Budget Remaining;
-   overspend;
-   budget pace;
-   projected spending;
-   projected overspend.

A Budget is a spending plan, not a reservation of money.

Spending Planning consumes actual spending facts from Financial Accounts
& Ledger but MUST NOT own or mutate Ledger transactions.

Transaction Category is owned by Financial Accounts & Ledger. Spending
Planning owns only the relationship between a Budget and applicable
Categories.

Therefore deleting a Budget MUST NOT imply deletion of its Categories.

Spending Planning is authoritative for budget performance, but it MUST
NOT independently decide whether the user's overall financial condition
is safe.

For example, it may determine that a Budget is ahead of pace; Financial
Intelligence may interpret that assessment together with other domains
to determine broader impact and guidance.

------------------------------------------------------------------------

### 5.5 Financial Commitments

**Primary question:**\
\> What future payments is the user committed to?

Conceptual ownership includes:

-   obligations;
-   recurring obligations;
-   credit-card statement payment obligations;
-   PayLater settlement obligations;
-   installment and repayment schedules;
-   loan/KPR payment schedules;
-   due dates;
-   funding state;
-   obligation lifecycle.

A financial account and a financial commitment are distinct concepts.

For example:

``` text
Credit Card Account
Outstanding = Rp12m
        │
        │ statement generated
        ▼
Payment Obligation
Rp8m due 5 Nov
```

The account answers the current outstanding position. The commitment
answers what must be paid and when.

#### Installment Origin

A Financial Commitment may originate from another domain.

A credit-card purchase converted to installments remains an actual
Ledger expense. The installment schedule represents future payment
commitments related to that purchase.

Repaying installments MUST NOT create the original expense again.

This principle also applies to PayLater installment behavior.

KPR/loan schedules may also create future payment commitments even
though their origin differs from a credit-card transaction.

Financial Commitments therefore owns **future payment obligations and
repayment schedules regardless of origin**, without claiming ownership
of the originating Account or Transaction.

------------------------------------------------------------------------

### 5.6 Financial Calendar

**Primary question:**  
> What financial period does a date belong to?

Conceptual ownership includes:

- payday anchor;
- payday adjustment policy;
- Financial Cycle;
- cycle resolution;
- authoritative financial temporal boundaries.

Other contexts MAY consume resolved Financial Cycle information but MUST NOT independently implement conflicting cycle formulas.

Financial Calendar MUST NOT understand domain concepts such as Budget, Salary, Safe-to-Spend, or Forecast merely because those domains use a Financial Cycle.

#### Mid-Cycle Configuration Changes

A payday / Financial Cycle configuration change made during an active cycle MUST NOT retroactively redefine that active cycle.

The new configuration becomes effective after the active cycle through an **Adjustment Cycle Strategy**.

The transition is determined from the boundary immediately following the current active cycle to the first regular boundary produced by the new payday anchor and its applicable payday-adjustment policy.

```text
Transition duration < 15 days
        │
        ▼
EXTENDED CYCLE
        │
        └── merge the transition period
            with the following regular cycle

Transition duration ≥ 15 days
        │
        ▼
SHORT CYCLE
        │
        └── keep the transition period
            as an independent Financial Cycle
            before regular new-anchor cycles begin
```

Therefore:

- the active Financial Cycle remains unchanged;
- a transition shorter than 15 days MUST NOT become an independent Financial Cycle;
- a transition of 15 days or longer MUST become an independent Short Cycle;
- after the transition is resolved, subsequent Financial Cycles follow the new regular anchor schedule.

Financial Calendar remains the sole authority for resolving these boundaries. Other contexts MUST consume the resulting cycle definition rather than independently reproduce the transition algorithm.

### 5.7 Financial Intelligence --- Core Bounded Context

**Primary question:**\
\> Given authoritative financial facts and assessments, where is the
user now, where are they heading, what could go wrong, and what should
they pay attention to?

Financial Intelligence consumes authoritative information from
Supporting Contexts and produces new cross-domain financial knowledge.

#### Financial Positioning

Financial Positioning owns Safe-to-Spend and Funding Deficit.

Safe-to-Spend is intentionally **not** owned by Money Allocation.

Financial Positioning MUST distinguish between **Funded Obligations** and **Unfunded Obligations**.

An Obligation, or relevant portion of an Obligation, is **Funded** when it is already protected by Reserved Money. A funded amount MUST NOT be deducted from Flexible Money again when calculating Safe-to-Spend.

An **Unfunded Obligation** is the applicable obligation amount that remains uncovered by Reserved Money. For a partially funded Obligation, only the uncovered portion is Unfunded.

```text
Unfunded Obligation Amount
=
max(
    0,
    Applicable Obligation Amount
    -
    Applicable Reserved Coverage
)
```

Safe-to-Spend follows:

```text
RawSTS
=
FlexibleMoney
-
ApplicableUnfundedObligationsWithinCurrentCycle

DisplayedSTS
=
max(0, RawSTS)

FundingDeficit
=
max(0, -RawSTS)
```

This establishes the **Obligation Deduction Prevention Invariant**:

> Money already removed from Flexible Money and protected as Reserved Money MUST NOT cause the same Obligation to reduce Safe-to-Spend a second time.

Safe-to-Spend is a cross-domain interpretation that no individual Supporting Context can authoritatively produce on its own.

#### Financial Projection

Financial Projection owns Base Forecast and Scenario Forecast within
Financial Intelligence.

It MUST preserve the locked Actual-vs-Forecast separation.

Forecast calculations MUST NOT mutate actual financial facts.

Expected stable income may participate in Base Forecast according to
Financial Engine rules.

Expected uncertain income MUST NOT automatically enter Base Forecast; it
may participate in explicitly selected scenarios.

#### Risk Detection

Risk Detection identifies deterministic financial conditions from
authoritative inputs and Financial Intelligence results.

Examples include:

-   Funding Dependency;
-   Low Safe-to-Spend;
-   cross-domain risk arising from budget performance, commitments,
    expected income, position, or projection.

Supporting Contexts retain ownership of their own assessments. Financial
Intelligence MUST NOT reimplement Supporting Context rules merely to
produce guidance.

#### Financial Guidance

Guidance is an essential Core Domain output.

The guidance structure follows:

``` text
WHAT HAPPENED
      ↓
WHY
      ↓
LIKELY HAPPENS
      ↓
IMPACT
      ↓
ACTION
```

Guidance MUST be explainable from deterministic financial evidence.

It MUST NOT depend on an LLM to establish financial truth or authorize
state changes.

------------------------------------------------------------------------

## 6. Domain Ownership Matrix

  -----------------------------------------------------------------------
  Concept                             Authoritative Bounded Context
  ----------------------------------- -----------------------------------
  Financial Account                   Financial Accounts & Ledger

  Cash / Bank / E-Wallet Account      Financial Accounts & Ledger

  Credit Card Account                 Financial Accounts & Ledger

  PayLater Account                    Financial Accounts & Ledger

  Actual Transaction                  Financial Accounts & Ledger

  Actual Income                       Financial Accounts & Ledger

  Actual Expense                      Financial Accounts & Ledger

  Transfer / Payment                  Financial Accounts & Ledger

  Transaction Category                Financial Accounts & Ledger

  Current Credit Outstanding          Financial Accounts & Ledger

  Expected Income                     Income Management

  Income Schedule                     Income Management

  Income Stability                    Income Management

  Expected-Income Lifecycle           Income Management

  Allocation                          Money Allocation

  Flexible / Reserved / Locked /      Money Allocation
  Unallocated                         

  Allocation Shortfall                Money Allocation

  Budget                              Spending Planning

  Budget ↔ Category Mapping           Spending Planning

  Budget Spent / Remaining / Pace     Spending Planning

  Projected Budget Spending /         Spending Planning
  Overspend                           

  Obligation                          Financial Commitments

  Credit Statement / PayLater         Financial Commitments
  Settlement Obligation               

  Installment / Repayment Schedule    Financial Commitments

  KPR / Loan Payment Schedule         Financial Commitments

  Financial Cycle                     Financial Calendar

  Payday Anchor / Adjustment Policy   Financial Calendar

  Safe-to-Spend                       **Financial Intelligence**

  Funding Deficit                     **Financial Intelligence**

  Base / Scenario Forecast            **Financial Intelligence**

  Funding Dependency                  **Financial Intelligence**

  Cross-domain Risk Detection         **Financial Intelligence**

  Financial Guidance                  **Financial Intelligence**
  -----------------------------------------------------------------------

This table defines conceptual authority, not database ownership or
package structure.

------------------------------------------------------------------------

## 7. Context Relationships

The high-level Context Map is:

``` text
                         ┌────────────────────┐
                         │ Financial Calendar │
                         └─────────┬──────────┘
                                   │
                         temporal context
                                   │
            ┌──────────────────────┼─────────────────────┐
            │                      │                     │
            ▼                      ▼                     ▼
┌──────────────────┐    ┌──────────────────┐   ┌───────────────────┐
│ Income           │    │ Money Allocation │   │ Spending Planning │
│ Management       │    └─────────▲────────┘   └─────────▲─────────┘
└────────▲─────────┘              │                      │
         │                        │                      │
 actual  │                  actual-money          spending facts
 income  │                     changes                  │
 facts   │                        │                      │
         │              ┌─────────┴──────────────────────┘
         │              │
         │     ┌────────┴───────────────┐
         └─────│ Financial Accounts &   │
               │ Ledger                 │
               └────────┬───────────────┘
                        │
                 financial activity
                        │
                        ▼
               ┌───────────────────────┐
               │ Financial Commitments │
               └──────────┬────────────┘
                          │
                          │ authoritative facts /
                          │ assessments
                          ▼
              ┌──────────────────────────────┐
              │ ★ Financial Intelligence    │
              │                              │
              │ Position                     │
              │    ↓                         │
              │ Project                      │
              │    ↓                         │
              │ Detect                       │
              │    ↓                         │
              │ Guide                        │
              └──────────────────────────────┘
```

The diagram communicates dependency direction, not implementation calls.

A Bounded Context MUST NOT directly manipulate another context's
internal domain model merely because a relationship exists.

------------------------------------------------------------------------

## 8. Integration Principles

### 8.1 Explicit Contracts Between Contexts

Contexts MUST communicate through explicit application/domain contracts
rather than directly depending on another context's internal entities,
repositories, persistence tables, or implementation details.

In the modular monolith, such contracts do not imply HTTP or network
APIs.

They may be in-process Kotlin interfaces, immutable snapshots,
application requests, domain events, or other explicitly designed
contracts.

Concrete contract shapes are deferred.

### 8.2 Queries and Events Serve Different Purposes

A synchronous query/contract answers:

> What is authoritative right now?

A domain event communicates:

> A meaningful domain fact has occurred.

The architecture MUST NOT force all cross-context communication through
events.

Likewise, domain events MUST NOT become a disguised Event Sourcing
mechanism.

### 8.3 Events Signal Change; Authoritative Contexts Provide State

Financial Intelligence SHOULD NOT reconstruct all current financial
truth solely by replaying domain events.

A preferred conceptual model is:

``` text
Domain Event
    │
    │ signals that relevant facts changed
    ▼
Dependent capability
    │
    │ obtains required authoritative snapshots/contracts
    ▼
Recalculation
```

This aligns with the Architecture Constitution principle:

> Domain events indicate what changed; the calculation dependency graph
> determines what must be recomputed.

------------------------------------------------------------------------

## 9. Financially Critical Cross-Context Operations

Financially critical operations require immediate consistency.

The agreed integration model is:

> Commands coordinate required synchronous domain work; events
> communicate facts and trigger appropriate reactions.

For example, conceptually:

``` text
RecordExpense Command
        │
        ▼
Application Orchestration
        │
        ├──► Financial Accounts & Ledger
        │      record actual expense
        │
        ├──► Spending Planning
        │      reflect/recalculate budget actual
        │
        ├──► Money Allocation
        │      reflect actual-money change
        │
        └──► Financial Intelligence
               recalculate critical financial position
        │
        ▼
      COMMIT
        │
        ▼
User sees successful save
        │
        ▼
Domain Events / Secondary Reactions
```

The exact transaction mechanism and concrete orchestration design are
deferred to later architecture/design stages.

The strategic rules are:

-   Financially critical state MUST be correct when the operation is
    reported successful.
-   Domain events MUST NOT be the sole correctness mechanism for
    critical financial state.
-   The Application Layer MAY coordinate multiple Bounded Contexts.
-   The Application Layer MUST NOT absorb business rules owned by those
    contexts.
-   Each Bounded Context remains responsible for enforcing its own
    domain rules.

------------------------------------------------------------------------

## 10. Key Cross-Context Relationships

### 10.1 Accounts & Ledger → Spending Planning

Financial Accounts & Ledger is upstream for actual spending facts.

Spending Planning consumes relevant spending information to
maintain/recalculate Budget performance.

Spending Planning MUST NOT own or mutate Ledger transactions.

Transaction edit, deletion, refund, and backdating MUST be able to
propagate their financial effect to dependent Budget calculations
according to the locked requirements.

Concrete domain-event names are deferred to the Event Catalog.

### 10.2 Accounts & Ledger → Income Management

Financial Accounts & Ledger is authoritative for actual received income.

Income Management is authoritative for expected-income lifecycle.

V1 uses an **Explicit Claim / Manual Link on Record** integration mechanism for expected-income realization.

When recording an actual Income transaction, the user MAY explicitly select an Expected Income occurrence that the transaction realizes.

```text
Record Income
     │
     ├── actual transaction ──► Accounts & Ledger
     │
     └── optional Expected Income reference
                         │
                         ▼
                  Income Management
                         │
                         ▼
                  EXPECTED → RECEIVED
```

The Ledger remains authoritative for whether money was actually received. Income Management remains authoritative for the lifecycle transition of Expected Income.

The association MUST originate from explicit user intent in V1. The system MUST NOT infer this association through fuzzy matching based on amount, date proximity, description, source name, or similar heuristics.

An actual Income transaction MAY remain unlinked to Expected Income. An Expected Income MUST NOT transition to `RECEIVED` merely because a similar actual Income exists.

### 10.3 Accounts & Ledger → Money Allocation

Actual new/liquid money becomes available for allocation according to
the locked product rules.

Financial Accounts & Ledger establishes that actual money changed.

Money Allocation establishes the role of that money.

Ledger MUST NOT decide how incoming money is divided between Flexible,
Reserved, or Locked roles.

### 10.4 Accounts & Ledger ↔ Financial Commitments

Financial activity can create or settle commitments.

Examples include:

-   a credit statement creating a payment obligation;
-   a PayLater settlement creating a payment obligation;
-   a credit purchase being structured into an installment schedule;
-   a payment reducing an outstanding obligation.

The Account/Transaction and the Commitment remain distinct authoritative
concepts even when one business operation affects both.

### 10.5 Supporting Contexts → Financial Intelligence

Financial Intelligence is primarily downstream of authoritative
Supporting Contexts.

It consumes financial facts and domain assessments to create
cross-domain interpretations.

It MUST NOT take ownership of Supporting Context concepts merely because
it uses them in calculations.

Examples:

-   it consumes Flexible Money; it does not own Allocation;
-   it consumes Budget assessment; it does not own Budget;
-   it consumes Expected Income; it does not own the expected-income
    lifecycle;
-   it consumes Obligations; it does not own obligation lifecycle;
-   it consumes Financial Cycle; it does not independently define cycle
    boundaries.

------------------------------------------------------------------------

## 11. Financial Intelligence Does Not Command Supporting Domains

Financial Intelligence advises; it does not autonomously modify
financial facts or plans.

The intended interaction is:

``` text
Supporting Contexts
        │
        │ authoritative facts / assessments
        ▼
Financial Intelligence
        │
        ├── Position
        ├── Project
        ├── Detect
        └── Guide
        │
        ▼
Recommendation / Explanation
        │
        ▼
User / Presentation
        │
        │ explicit user intent
        ▼
Application Layer
        │
        ▼
Owning Bounded Context
```

Example --- Funding Dependency:

``` text
Financial Intelligence
        │
        ▼
Recommendation:
"Reserve money for upcoming KPR"
        │
        ▼
UI: [Reserve Money]
        │
        │ user explicitly chooses action
        ▼
Application Layer
        │
        ▼
Money Allocation
```

Financial Intelligence MUST NOT directly reserve the money.

This avoids circular domain authority and preserves human agency over
financial state changes.

------------------------------------------------------------------------

## 12. AI Boundary

Future LLM functionality MUST preserve the same domain boundary.

For natural-language transaction capture:

``` text
Natural Language
      ↓
LLM / Intent Parser
      ↓
Structured Transaction Draft
      ↓
Deterministic Validation
      ↓
User Confirmation
      ↓
Application Command
      ↓
Owning Domain
```

An LLM MAY propose structured intent or explain deterministic financial
results.

An LLM MUST NOT:

-   become the source of financial truth;
-   directly modify financial state;
-   bypass deterministic domain validation;
-   independently calculate authoritative financial results;
-   automatically execute a Financial Intelligence recommendation
    without the required user intent.

------------------------------------------------------------------------

## 13. Ubiquitous Language Boundaries

The following distinctions MUST remain explicit.

### Account vs Commitment

An Account represents a financial account/facility and its current state.

A Commitment represents a future payment obligation.

```text
Credit Card Outstanding ≠ Statement Amount Due
```

### Transaction vs Commitment

A Transaction represents an actual financial event.

A Commitment represents something that must be paid.

```text
Original Credit Purchase Expense ≠ Installment Repayment
```

### Budget vs Allocation

A Budget represents a spending plan.

Allocation represents the role assigned to actual money.

```text
Creating Budget ≠ Reserving Money
```

### Expected Income vs Actual Income

Expected Income represents anticipated inflow.

Actual Income is a Ledger fact.

```text
Expected Income ≠ Actual Money
```

An Expected Income becomes `RECEIVED` in V1 only when an actual Income transaction is explicitly claimed/linked to it through user intent.

```text
Actual Income exists
        ≠
Expected Income automatically RECEIVED
```

### Flexible Money vs Reserved Money

Flexible Money represents actual money available for discretionary financial use subject to Financial Positioning rules.

Reserved Money represents actual money already protected for a specific purpose or obligation.

Money moved from Flexible into Reserved is no longer part of Flexible Money.

### Funded Obligation vs Unfunded Obligation

A **Funded Obligation** is an applicable Obligation, or portion thereof, already protected by Reserved Money.

An **Unfunded Obligation** is the remaining applicable obligation amount not protected by Reserved Money.

```text
Applicable Obligation
        -
Applicable Reserved Coverage
        =
Unfunded Obligation
```

For partially protected obligations, only the uncovered portion is Unfunded.

### Flexible Money vs Safe-to-Spend

Flexible Money is allocation state.

Safe-to-Spend is a Financial Intelligence result after considering applicable **Unfunded Obligations** and Financial Cycle rules.

```text
Flexible Money ≠ Safe-to-Spend
```

Safe-to-Spend MUST NOT deduct an Obligation amount already protected by Reserved Money.

### Account Outstanding vs Future Cash Flow

Current liability outstanding does not by itself define the future payment schedule.

Financial Commitments provides the authoritative future obligation schedule used by forecasting.

### Active Cycle vs Adjustment Cycle

The **Active Cycle** is the currently effective Financial Cycle and MUST NOT be retroactively redefined by a payday-anchor configuration change.

An **Adjustment Cycle** is the transition mechanism between the current cycle schedule and a newly configured anchor.

An Adjustment Cycle may resolve as:

- a **Short Cycle** when the transition duration is at least 15 days; or
- part of an **Extended Cycle** when the transition duration is less than 15 days.

## 14. Strategic Invariants

The following strategic rules constrain later Tactical DDD and implementation design:

1. Each major business concept MUST have one authoritative Bounded Context.
2. A downstream context MUST NOT duplicate the authoritative business rules of an upstream context.
3. Financial Intelligence MUST create cross-domain interpretation rather than absorb ownership of every input domain.
4. Financial Intelligence MUST remain deterministic for authoritative financial calculations and rules.
5. Financial Intelligence MUST NOT autonomously mutate Supporting Context state.
6. Actual and forecast financial state MUST remain separate.
7. Expected Income MUST NOT become actual money without an actual Ledger fact.
8. Expected Income MUST NOT transition to `RECEIVED` through fuzzy or heuristic matching in V1; realization requires an explicit user claim/link to an actual Income transaction.
9. Budget MUST remain independent from money reservation/allocation.
10. Repayment of a previously recognized credit expense MUST NOT recognize that expense again.
11. Account state and future payment obligations MUST remain distinct concepts.
12. Financial Cycle calculation MUST have a single authoritative definition.
13. A mid-cycle anchor change MUST NOT retroactively redefine the active Financial Cycle.
14. Financial Calendar MUST resolve anchor changes through the approved Adjustment Cycle Strategy.
15. A transition duration below 15 days MUST be merged into an Extended Cycle.
16. A transition duration of 15 days or more MUST form an independent Short Cycle before regular cycles under the new anchor.
17. Safe-to-Spend MUST deduct only Applicable Unfunded Obligations within the current Financial Cycle.
18. An Obligation amount already protected by Reserved Money MUST NOT reduce Flexible Money again in Safe-to-Spend.
19. A partially funded Obligation MUST contribute only its uncovered amount to Safe-to-Spend deduction.
20. Financially critical cross-context operations MUST achieve immediate consistency before success is presented.
21. Application orchestration MUST NOT become a substitute for domain ownership.
22. Cross-context communication MUST use explicit contracts rather than internal persistence/model access.
23. Domain events MUST represent meaningful facts and MUST NOT turn the system into Event Sourcing by accident.
24. Financial Intelligence MUST recommend rather than autonomously execute user financial actions.

## 15. Explicit Non-Decisions in v1.0

The following are intentionally **not** decided by this Strategic DDD document:

- Aggregate Roots;
- Entity and Value Object boundaries;
- exact class/interface names;
- Kotlin module/package layout;
- repository interfaces;
- database tables and relationships;
- SQLDelight schema;
- exact transaction boundary implementation;
- exact domain-event names and payloads;
- complete Event Catalog;
- command/query catalog;
- concrete read models;
- exact persistence representation of the Explicit Claim / Manual Link relationship;
- exact representation of Credit Card / PayLater specialized account terms;
- exact installment model hierarchy;
- calculation dependency graph;
- persistence/cache strategy beyond the Architecture Constitution;
- future multi-device synchronization protocol.

The following items are **no longer non-decisions** and are frozen by this document:

- Expected Income realization uses Explicit Claim / Manual Link on Record in V1.
- Automatic fuzzy matching for Expected Income realization is excluded from V1.
- Mid-cycle payday-anchor changes use the Adjustment Cycle Strategy.
- Adjustment transitions below 15 days merge into an Extended Cycle.
- Adjustment transitions of 15 days or more become an independent Short Cycle.
- Safe-to-Spend deducts only Applicable Unfunded Obligations.
- Reserved coverage prevents duplicate obligation deduction from Flexible Money.

Remaining non-decisions belong to Tactical DDD, calculation design, persistence design, or technical architecture phases.

## 16. Resolved Review Clarifications

All blocking Strategic DDD review issues have been resolved for v1.0.

### 16.1 Financial Cycle Anchor Change — RESOLVED

1. A payday-anchor change made during an active Financial Cycle MUST NOT alter that active cycle.
2. Financial Calendar resolves the transition after the current cycle using the new anchor and applicable payday-adjustment policy.
3. The transition duration is measured from the boundary following the active cycle to the first regular boundary under the new anchor.
4. If the resulting transition duration is **less than 15 days**, it MUST NOT become a standalone cycle. It is merged with the following regular cycle to create an **Extended Cycle**.
5. If the resulting transition duration is **15 days or more**, it becomes an independent **Short Cycle**.
6. After the adjustment period, regular Financial Cycles follow the new anchor schedule.

This clarification has been reconciled into the appropriate Product Requirements and Financial Engine documentation.

### 16.2 Expected Income Realization — RESOLVED

Financial Accounts & Ledger owns Actual Income. Income Management owns Expected Income and its lifecycle.

V1 realizes Expected Income through **Explicit Claim / Manual Link on Record**. When recording an actual Income transaction, the user MAY explicitly link it to a relevant Expected Income occurrence.

A successful explicit link allows Income Management to transition:

```text
EXPECTED → RECEIVED
```

No fuzzy matching is permitted in V1. Similarity in amount, date, description, or income source MUST NOT independently cause realization.

### 16.3 Obligation Deduction Prevention — RESOLVED

Only Applicable Unfunded Obligations within the current Financial Cycle reduce Flexible Money when calculating Safe-to-Spend.

```text
Unfunded Obligation Amount
=
max(0, Applicable Obligation Amount - Applicable Reserved Coverage)

RawSTS
=
FlexibleMoney
-
ApplicableUnfundedObligationsWithinCurrentCycle
```

Reserved coverage therefore prevents duplicate deduction.

### 16.4 Context Boundary Validation — RESOLVED

The seven Bounded Contexts are approved for V1:

1. Financial Accounts & Ledger
2. Income Management
3. Money Allocation
4. Spending Planning
5. Financial Commitments
6. Financial Calendar
7. Financial Intelligence

Income Management and Financial Commitments remain separate because their ubiquitous language, lifecycle, and authority differ.

### 16.5 Financial Intelligence Cohesion — RESOLVED

Positioning, Projection, Risk Detection, and Guidance remain cohesive internal capabilities of one Financial Intelligence Bounded Context for V1.

They MUST NOT be split into independent Bounded Contexts merely for architectural symmetry.

## 17. Context Map Summary

``` text
                         FINANCIAL CALENDAR
                         temporal authority
                                │
                ┌───────────────┼───────────────┐
                │               │               │
                ▼               ▼               ▼
          INCOME MGMT     MONEY ALLOCATION   SPENDING PLANNING
                ▲               ▲               ▲
                │               │               │
                │               │               │
                └───────┐       │       ┌───────┘
                        │       │       │
                  FINANCIAL ACCOUNTS & LEDGER
                  actual financial authority
                        │
                        ▼
                FINANCIAL COMMITMENTS
                future obligation authority

          All relevant authoritative facts / assessments
                              │
                              ▼
                 ★ FINANCIAL INTELLIGENCE
                 ┌────────────────────────┐
                 │ Position               │
                 │    ↓                   │
                 │ Project                │
                 │    ↓                   │
                 │ Detect                 │
                 │    ↓                   │
                 │ Guide                  │
                 └────────────────────────┘
                              │
                              ▼
                    explanation / action
                         recommendation
                              │
                              ▼
                             USER
                              │
                        explicit intent
                              ▼
                     APPLICATION LAYER
                              │
                              ▼
                    OWNING BOUNDED CONTEXT
```

The map is directional but intentionally conceptual. It does not
prescribe physical deployment boundaries.

------------------------------------------------------------------------

## 18. Approval Checklist

Strategic DDD v1.0 has been reviewed against the following decisions:

- [x] Financial Intelligence is the Core Domain.
- [x] Guidance is treated as an essential Financial GPS output.
- [x] The six Supporting Subdomains are correctly identified.
- [x] Seven Bounded Contexts are approved for V1.
- [x] Every major business concept has a clear authoritative owner.
- [x] Category belongs to Financial Accounts & Ledger, while Budget-to-Category mapping belongs to Spending Planning.
- [x] Budget and Allocation remain independent concepts.
- [x] Safe-to-Spend belongs to Financial Intelligence / Financial Positioning.
- [x] Flexible Money and Safe-to-Spend remain distinct concepts.
- [x] Funded and Unfunded Obligations are explicitly distinguished for Safe-to-Spend.
- [x] Reserved Money prevents duplicate obligation deduction from Flexible Money.
- [x] Partially funded Obligations contribute only their uncovered amount to Safe-to-Spend deduction.
- [x] Expected Income and Actual Income remain separate.
- [x] Expected Income realization uses Explicit Claim / Manual Link on Record.
- [x] Automatic fuzzy matching of actual and Expected Income is excluded from V1.
- [x] Credit Card and PayLater are modeled strategically as financial accounts/facilities.
- [x] Credit statement/settlement obligations belong to Financial Commitments.
- [x] Installment schedules belong to Financial Commitments regardless of their origin.
- [x] Credit repayment does not create duplicate expense recognition.
- [x] Financial Calendar is the single authority for Financial Cycle boundaries.
- [x] Mid-cycle cycle-configuration changes do not retroactively redefine the active cycle.
- [x] Financial Cycle anchor transitions use the Adjustment Cycle Strategy.
- [x] Transition periods below 15 days are merged into an Extended Cycle.
- [x] Transition periods of 15 days or more become an independent Short Cycle.
- [x] Financially critical cross-context work is synchronously coordinated before success is presented.
- [x] Domain events complement rather than replace critical synchronous coordination.
- [x] Financial Intelligence consumes authoritative facts without taking ownership of Supporting Context rules.
- [x] Financial Intelligence recommends actions but does not autonomously execute them.
- [x] User intent remains the boundary before recommendations modify financial state.
- [x] Income Management and Financial Commitments remain separate Bounded Contexts.
- [x] Positioning, Projection, Risk Detection, and Guidance remain cohesive capabilities within one Financial Intelligence Bounded Context.
- [x] No Tactical DDD or technical implementation detail has been prematurely locked.
- [x] All blocking Strategic DDD clarification items have been resolved.
- [x] Remaining intentionally deferred decisions are explicitly documented.

## 19. Next Step

Strategic DDD is now frozen at v1.0. The next design stage is **Tactical DDD**.

``` text
Strategic DDD v1.0
  APPROVED / FROZEN
        │
        ▼
Tactical DDD
        │
        ├── Aggregates
        ├── Entities
        ├── Value Objects
        ├── Invariants
        ├── Commands
        └── Domain Events
```

Tactical DDD MUST conform to this approved and frozen Strategic DDD
model and MUST NOT redefine its strategic decisions.
