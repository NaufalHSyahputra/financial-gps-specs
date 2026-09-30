# Financial Calendar — Tactical DDD v1.0

**Status:** FROZEN SPECIFICATION v1.0  
**Bounded Context:** Financial Calendar  
**Decision range:** TD-CAL-001 through TD-CAL-048  
**Purpose:** Define the frozen Tactical DDD specification for the Financial Calendar bounded context, consolidating TD-CAL-001 through TD-CAL-048 and the resolved audit findings without introducing behavior beyond the approved clarifications.  
**Governance:** This document is subordinate to the frozen Product Requirements Baseline, Financial Engine, Architecture Constitution, and Strategic DDD. Any conflict with those sources must be resolved by review/change control rather than silently overridden here.

---

## 1. Responsibility and Boundary

Financial Calendar is the sole domain authority for Financial Cycle boundaries and cycle-relative temporal semantics. It owns:

- Payday Anchor and weekday Adjustment Policy semantics;
- deterministic Financial Cycle resolution;
- preservation of the currently resolved cycle when configuration changes;
- transition classification into `REGULAR`, `SHORT`, or `EXTENDED` cycles;
- effective configuration history required to reconstruct Financial Cycles;
- cycle-relative position (`TotalCycleDays`, `ElapsedCycleDays`, `RemainingCycleDays`).

Financial Calendar does **not** own Budget, Salary/Expected Income, Safe-to-Spend, Forecast, transaction classification, obligations, or other financial calculations merely because those domains consume a Financial Cycle.

Other bounded contexts MUST consume Financial Calendar's authoritative resolved cycle information rather than independently implementing cycle formulas.

---

## 2. Core Tactical Model

```text
FinancialCalendarConfiguration                 <<Aggregate Root>>
│
├── CalendarConfigurationId
├── InitializedAt
├── EffectiveRevisions
│   └── ConfigurationRevision                  <<Entity>>
│       ├── RevisionId
│       ├── Settings                           <<Value Object>>
│       │   ├── PaydayAnchor                   <<Value Object>>
│       │   └── AdjustmentPolicy               <<Enum / Value>>
│       └── EffectiveFrom                      <<FinancialDate>>
│
└── PendingConfigurationChange?                <<Value Object>>
    ├── RequestedSettings
    ├── RequestedOn                            <<FinancialDate>>
    ├── OccurredAt                             <<Instant>>
    └── ProtectedThrough                       <<FinancialDate>>

FinancialCycle                                 <<Derived Value Object>>
├── DateRange
│   ├── StartDate
│   └── EndDate
└── CycleType: REGULAR | SHORT | EXTENDED

FinancialCyclePosition                         <<Derived Value Object>>
├── AsOfDate
├── TotalCycleDays
├── ElapsedCycleDays
└── RemainingCycleDays

FinancialCycleResolver                         <<Pure Domain Service>>
```

`ActiveConfiguration` is deliberately not an independently persisted authoritative pointer. Effective configuration is determined from temporal facts and the requested date.

---

## 3. Aggregate Root

### 3.1 `FinancialCalendarConfiguration`

`FinancialCalendarConfiguration` is the Aggregate Root responsible for protecting the invariants of the configuration timeline.

Its responsibilities include:

- initialization exactly once;
- accepting a requested configuration change;
- protecting the entire Financial Cycle active at the time of a change;
- replacing a still-non-effective pending change;
- cancelling a still-non-effective pending change when the user returns to the effective settings;
- preserving effective configuration history once a configuration has become temporally effective;
- ensuring configuration changes are evaluated against logical temporal state, not stale persistence labels.

The Aggregate Root does **not** create, close, activate, or persist canonical Financial Cycle instances.

---

## 4. Entities and Value Objects

### 4.1 `ConfigurationRevision` — Entity

An effective configuration revision has historical identity. Two revisions may contain identical settings but still represent different effective periods.

Conceptual fields:

```text
ConfigurationRevision
├── RevisionId
├── Settings
└── EffectiveFrom
```

Example: Anchor 10 may govern January, change to 15 in March, then return to 10 in June. The January and June revisions are not the same Entity even though their settings are value-equal.

### 4.2 `FinancialCalendarSettings` — Value Object

Conceptually contains:

```text
FinancialCalendarSettings
├── PaydayAnchor
└── AdjustmentPolicy
```

`PaydayAnchor` is valid only in `1..31`.

`AdjustmentPolicy` is one of:

- `NONE`
- `PREVIOUS_WORKING_DAY`
- `NEXT_WORKING_DAY`

Timezone is intentionally not embedded in this settings Value Object by the current Tactical decision. Timezone participates at the boundary that converts a current instant to a local financial date.

### 4.3 `PendingConfigurationChange` — Value Object

A non-effective pending change has no independent domain identity. It can be replaced while it has not yet become temporally effective.

Conceptual fields:

```text
PendingConfigurationChange
├── RequestedSettings
├── RequestedOn        <<FinancialDate>>
├── OccurredAt         <<Instant>>
└── ProtectedThrough   <<FinancialDate>>
```

`RequestedOn` is the business financial date used to evaluate the effective configuration, resolve the currently protected Financial Cycle, and apply Financial Calendar rules. `OccurredAt` is the audit/event occurrence timestamp representing when the user/system operation occurred in absolute time.

`OccurredAt` MUST NOT implicitly determine a Financial Cycle boundary or business financial date. Any use of an `Instant` for Financial Calendar business semantics requires explicit conversion through the configured `FinancialTimezone` into a `FinancialDate` at the Application Service / Use Case boundary.

Audit history may retain superseded user actions, but superseded pending choices do not become effective domain revisions.

### 4.4 `FinancialCycle` — Derived Value Object

`FinancialCycle` is an immutable derived domain value, not a persisted canonical historical Entity.

```text
FinancialCycle
├── StartDate
├── EndDate
└── CycleType
```

It has no independent `FinancialCycleId`. Value equality is based on its contained values.

`CycleType` is authoritative domain output from Financial Calendar and MUST NOT be independently inferred by consumers.

### 4.5 `FinancialCyclePosition` — Derived Value Object

Cycle-relative position is separate from the immutable cycle because it depends on an `AsOfDate`.

```text
FinancialCyclePosition
├── AsOfDate
├── TotalCycleDays
├── ElapsedCycleDays
└── RemainingCycleDays
```

It is derived, never progressed as mutable daily state.

---

## 5. Temporal and Configuration Lifecycle

### 5.1 Derived cycles, canonical configuration facts

Financial Cycles are resolved from configuration facts/history. Canonical state stores only the facts needed to deterministically reconstruct the calendar timeline; it does not pre-generate or persist every Financial Cycle as source of truth.

Core determinism property:

```text
same requested date
+ same effective configuration facts
= same FinancialCycle
```

### 5.2 Protected active cycle

When configuration changes during the cycle containing the change date, that entire resolved cycle is protected and MUST NOT be retroactively redefined.

`ProtectedThrough` is the inclusive end date of that protected cycle.

### 5.3 Effective date of a change

A requested configuration becomes temporally effective immediately after the protected cycle:

```text
EffectiveFrom = ProtectedThrough + 1 calendar day
```

Configuration effectiveness is not the same as the first regular payday boundary. The first cycle under the new configuration may be `SHORT` or `EXTENDED`.

### 5.4 Temporal promotion requires no scheduler

Passage of time alone is sufficient to change the logical interpretation of a stored pending change. Financial Calendar correctness MUST NOT depend on a midnight job, background scheduler, or mutation that flips a persisted `PENDING` status to `ACTIVE`.

`PENDING` / `ACTIVE` are therefore not authoritative persisted lifecycle truth.

### 5.5 Last pending configuration wins — only before effectiveness

A new requested configuration MAY replace an existing pending configuration only while the previous pending configuration has not yet become temporally effective.

Once temporally effective, the previous configuration is part of effective domain history. A subsequent user change is a new configuration change and protects the cycle that is active at that later change date.

### 5.6 Materialization is bookkeeping

If persistence later normalizes a temporally effective pending representation into an effective revision representation, that normalization is not a new business fact and does not emit a Domain Event.

---

## 6. Initial Configuration and Historical Floor

Initial setup is not treated as a configuration transition because no prior Financial Calendar history exists to protect.

Given an initialization date, the initial settings may resolve the Financial Cycle containing that date even when the cycle starts before application/configuration creation.

Example:

```text
InitializedAt = 20 Sep
Anchor = 10

Initial cycle = 10 Sep .. 9 Oct
Initial revision EffectiveFrom = 10 Sep
```

`InitializedAt` and `EffectiveFrom` have different meanings:

- `InitializedAt`: when the calendar was actually configured;
- `EffectiveFrom`: earliest date from which the initial configuration is authoritative for cycle resolution.

The first revision's `EffectiveFrom` is the historical floor. Financial Calendar MUST NOT extrapolate the initial configuration backward before that date.

A request earlier than the historical floor produces `CalendarHistoryUnavailable`, not an invented cycle.

---

## 7. Regular Boundary Resolution

Regular boundary resolution follows this exact deterministic order:

```text
PaydayAnchor
    ↓
short-month clamp
    ↓
Nominal Boundary
    ↓
weekday AdjustmentPolicy
    ↓
Effective Regular Boundary
```

### 7.1 Short-month clamping

For a year/month and `PaydayAnchor D`:

```text
nominalDay = min(D, lastDayOfMonth(year, month))
```

Examples:

- Anchor 31 in April → nominal 30 April;
- Anchor 31 in February → nominal 28/29 February;
- Anchor 30 in February → nominal 28/29 February;
- Anchor 29 in a non-leap February → nominal 28 February.

### 7.2 Weekday adjustment

V1 working day means Monday–Friday only. Public holidays are not modeled.

- `NONE`: effective boundary = nominal boundary.
- `PREVIOUS_WORKING_DAY`: weekend nominal boundary moves backward to the preceding weekday.
- `NEXT_WORKING_DAY`: weekend nominal boundary moves forward to the following weekday.

An adjustment MAY cross a calendar-month boundary. The adjusted/effective boundary remains authoritative.

### 7.3 Boundary monotonicity expectation

Consecutive regular effective boundaries must remain strictly ordered. This is an important candidate for property-based testing across years, anchors `1..31`, leap years, and all adjustment policies.

---

## 8. Adjustment Cycle Strategy

When a configuration change takes effect after `ProtectedThrough`:

1. Set transition start to the calendar day immediately after `ProtectedThrough`.
2. Resolve the first **adjusted/effective regular boundary** under the new settings.
3. Measure the transition duration in local financial dates from transition start through the date immediately preceding that regular boundary.
4. Classify the transition using the frozen threshold.

### 8.1 Short Cycle

If transition duration is **15 days or more**:

```text
transition start .. day before first new regular boundary
= SHORT cycle
```

The following cycle begins at the first new regular boundary and is regular.

Exactly 15 days is `SHORT`.

### 8.2 Extended Cycle

If transition duration is **less than 15 days**, the transition MUST NOT become an independent cycle. It is merged with the following regular cycle:

```text
transition start .. day before second new regular boundary
= EXTENDED cycle
```

### 8.3 Adjustment before classification

Weekday adjustment is applied before calculating the `<15` / `>=15` transition classification. A weekend adjustment can therefore legitimately change whether the resulting cycle is `SHORT` or `EXTENDED`.

### 8.4 Protected cycle may itself be non-regular

“Active cycle” means the cycle resolved for the change date, regardless of whether its type is `REGULAR`, `SHORT`, or `EXTENDED`. If the user changes configuration while in an `EXTENDED` or `SHORT` cycle, that entire cycle is protected before the next transition is calculated.

---

## 9. Local Date and Timezone Semantics

Financial Calendar's pure cycle model uses local financial calendar dates rather than UTC timestamps as the values of `StartDate`, `EndDate`, `EffectiveFrom`, `ProtectedThrough`, and `AsOfDate`.

`FinancialTimezone` is managed by the **Application Service / Use Case layer**, outside the Financial Calendar Aggregate and outside `FinancialCycleResolver`. The Application Service is responsible for reading the system clock, obtaining the configured `FinancialTimezone`, converting the current/evaluation instant into a local `FinancialDate`, and only then invoking the pure domain resolver:

```text
System Clock → Current Instant
                    +
          configured FinancialTimezone
                    ↓
              FinancialDate
                    ↓
          FinancialCycleResolver
          (pure domain service)
```

`FinancialCycleResolver` itself MUST NOT read the system clock, device timezone, own `FinancialTimezone`, or perform hidden UTC conversion.

`FinancialCycle.StartDate` and `FinancialCycle.EndDate` remain inclusive **Local Date** values. When the Financial Engine or another consumer requires an instant/timestamp range, the Application/Integration boundary MUST convert the cycle using the configured `FinancialTimezone` as a half-open range:

```text
InstantRange =
[start-of-day(StartDate, FinancialTimezone),
 start-of-day(EndDate + 1 day, FinancialTimezone))
```

Equivalently, the lower bound is inclusive and the upper bound is exclusive. UTC instants MUST NOT be stored inside `FinancialCycle` merely to support this integration representation.

Changing the configured timezone MUST NOT rewrite historical Financial Cycle date boundaries. It changes the derivation of the current local financial date and any required Local-Date-to-Instant conversion for subsequent time-sensitive operations.

Transaction timestamp/date reinterpretation after timezone changes is outside this bounded context and must be decided by Financial Accounts & Ledger.

---

## 10. Financial Cycle Position

For an inclusive cycle range and an `AsOfDate` inside that range:

```text
TotalCycleDays     = inclusive days(StartDate, EndDate)
ElapsedCycleDays   = inclusive days(StartDate, AsOfDate)
RemainingCycleDays = inclusive days(AsOfDate, EndDate)
```

Therefore:

```text
ElapsedCycleDays + RemainingCycleDays = TotalCycleDays + 1
```

The `+1` exists because `AsOfDate` belongs to both elapsed and remaining ranges.

On the final cycle day:

```text
RemainingCycleDays = 1
```

### 10.1 Range invariant

`FinancialCyclePosition` may be calculated only when:

```text
Cycle.StartDate <= AsOfDate <= Cycle.EndDate
```

An out-of-range date MUST NOT be silently clamped. It produces an explicit domain rejection such as `AsOfDateOutsideFinancialCycle`.

This rejection is distinct from `CalendarHistoryUnavailable`.

---

## 11. Commands and Business Outcomes

### 11.1 Minimal V1 mutation command surface

```text
InitializeFinancialCalendar
ChangeFinancialCalendarConfiguration
```

The following are intentionally **not** business commands:

- `CreateFinancialCycle`
- `UpdateFinancialCycle`
- `DeleteFinancialCycle`
- `ActivatePendingConfiguration`
- `GenerateNextCycle`
- `CloseCurrentCycle`

Cycles and temporal effectiveness are derived.

### 11.2 `InitializeFinancialCalendar`

Conceptual input:

```text
InitializeFinancialCalendar(
    settings,
    initializedOn
)
```

Behavior:

1. reject if already initialized;
2. validate/create settings Value Objects;
3. resolve the cycle containing `initializedOn`;
4. create initial `ConfigurationRevision`;
5. set first `EffectiveFrom` to that cycle's start date;
6. record `InitializedAt` separately.

### 11.3 `ChangeFinancialCalendarConfiguration`

Conceptual input:

```text
ChangeFinancialCalendarConfiguration(
    requestedSettings,
    requestedOn
)
```

The command MUST evaluate logical effective/pending state as of `requestedOn`; it must not trust stale physical `ACTIVE/PENDING` labels.

It resolves the cycle containing `requestedOn`, protects that entire cycle, and establishes/replaces/cancels the pending configuration according to the rules above.

### 11.4 Explicit business outcomes

Meaningful outcomes include:

- `Scheduled`
- `NoChange`
- `PendingChangeReplaced`
- `PendingChangeCancelled`

`NoChange` occurs when requested settings equal effective settings and there is no meaningful pending difference.

If a still-non-effective pending change exists and the user requests the currently effective settings, the pending change is cancelled rather than creating another change.

---

## 12. Queries and Output Contracts

### 12.1 `ResolveFinancialCycle(requestedDate)`

Authoritative cycle query.

Conceptual result:

```text
CycleResolutionResult
├── Resolved(FinancialCycle)
└── CalendarHistoryUnavailable
```

A valid out-of-history request is not represented by ambiguous `null` and is not a technical exception.

### 12.2 `GetEffectiveCalendarConfiguration(requestedDate)`

Returns a purpose-specific read contract such as:

```text
EffectiveCalendarConfiguration
├── PaydayAnchor
├── AdjustmentPolicy
└── EffectiveFrom
```

It does not expose a mutable Aggregate or require consumers to understand revision identity.

### 12.3 `GetPendingCalendarConfiguration(asOfDate)`

Returns a pending change only if it is still logically pending as of the supplied date. A physically unnormalized record that is already temporally effective MUST NOT be reported as pending.

### 12.4 `ResolveFinancialCycleContext(requestedDate)`

Financial Calendar MAY expose a coherent purpose-specific query returning:

```text
FinancialCycleContext
├── Cycle: FinancialCycle
└── Position: FinancialCyclePosition
```

Both values are resolved for the same requested date, reducing opportunities for consumers to combine mismatched temporal values.

### 12.5 No hidden “current” primitive

Primitive domain queries accept explicit `requestedDate` / `asOfDate`.

A convenience use case such as `GetCurrentFinancialCycle` belongs outside the pure resolver and performs:

```text
Clock.now()
+ configured Financial Timezone
→ FinancialDate
→ ResolveFinancialCycle(date)
```

### 12.6 No generic revision-history integration contract

Other bounded contexts MUST NOT receive a generic `GetAllConfigurationRevisions()` contract merely to reconstruct cycles themselves. A dedicated settings/history read model may be introduced only if an explicit UX/product requirement needs it.

---

## 13. Domain Services

### 13.1 `FinancialCycleResolver`

Pure domain service responsible for deterministic cycle resolution.

Conceptual form:

```text
resolve(
    requestedDate,
    configurationTimeline
) -> CycleResolutionResult
```

Properties:

- no database access;
- no repository access;
- no system clock access;
- no hidden timezone/device-time lookup;
- no mutation;
- deterministic for the same inputs.

It is the sole place where Financial Calendar interprets effective configuration history and Adjustment Cycle Strategy to produce authoritative cycle boundaries/type.

### 13.2 Cycle-position derivation

Cycle position is a pure derivation from `FinancialCycle + AsOfDate`. It may be implemented as behavior on a Value Object or as a small pure domain function/service; exact class placement is implementation detail so long as the ownership and invariants remain in Financial Calendar.

---

## 14. Domain Events

Domain Events represent meaningful domain mutations caused by operations. Passage of time by itself does not emit an event.

Minimal V1 event set:

- `FinancialCalendarInitialized`
- `PendingCalendarConfigurationScheduled`
- `PendingCalendarConfigurationReplaced`
- `PendingCalendarConfigurationCancelled`

Exact event payload schemas remain Event Catalog work.

### 14.1 No activation event required

Events such as `CalendarConfigurationActivated` / `BecameEffective` are not required for correctness. Effectiveness is derived from temporal facts.

Secondary features that need reminders/notifications may schedule their own reactions from the scheduling fact, but Financial Calendar correctness must not depend on them.

### 14.2 No event for no-op

`NoChange` produces no Financial Calendar Domain Event.

### 14.3 Materialization does not emit an event

Loading, normalizing, migrating, caching, or re-persisting state does not independently create a business Domain Event.

---

## 15. Failure Semantics

Financial Calendar distinguishes at least three categories.

### 15.1 Invalid input / Value Object construction failure

Examples:

- `PaydayAnchor = 0`
- `PaydayAnchor = 32`

Invalid values should be rejected at the input/Value Object boundary so invalid domain objects do not enter the Aggregate.

### 15.2 Domain rejection

The request is technically well-formed but violates a domain invariant.

Examples:

- initializing an already initialized Financial Calendar;
- calculating `FinancialCyclePosition` with `AsOfDate` outside the supplied cycle.

Candidate explicit result: `AsOfDateOutsideFinancialCycle`.

### 15.3 Valid domain/query outcome

The request is valid and the absence itself is meaningful.

Example:

```text
ResolveFinancialCycle(date before first revision EffectiveFrom)
→ CalendarHistoryUnavailable
```

This is not a technical failure and must not be silently converted to an invented cycle.

---

## 16. Cross-Context Contracts

Financial Calendar provides authoritative temporal facts to consumers such as Spending Planning, Financial Commitments, and Financial Intelligence.

Consumers MUST NOT independently redefine:

- cycle start/end boundaries;
- weekend adjustment;
- short-month clamping;
- Short/Extended transition classification;
- elapsed/remaining cycle-day semantics.

Financial Calendar does not learn consumer-domain concepts merely because those consumers use its temporal output.

Typical direction:

```text
Financial Calendar
    ↓ FinancialCycle / FinancialCycleContext
Spending Planning
Financial Commitments
Financial Intelligence
```

---

## 17. Persistence-Neutral Constraints

This Tactical model intentionally does not decide:

- SQLDelight table layout;
- whether effective revisions and transition facts share a table;
- how a temporally effective pending representation is physically normalized;
- repository interface signatures;
- Kotlin package/module structure;
- cache strategy;
- exact event serialization/payloads.

Persistence MUST preserve enough canonical facts to reproduce the same effective configuration timeline and resolved Financial Cycles deterministically.

Repository/persistence housekeeping MUST NOT invent Domain Events.

---

## 18. Consolidated Invariants

1. Financial Calendar is the sole authority for Financial Cycle boundaries and cycle-relative temporal semantics.
2. Financial Cycles are derived values, not canonical persisted historical Entities.
3. The same authoritative configuration facts and requested date produce the same Financial Cycle.
4. A configuration change never retroactively changes the Financial Cycle containing the change date.
5. The protected cycle may be `REGULAR`, `SHORT`, or `EXTENDED`.
6. A new configuration is effective from `ProtectedThrough + 1 day`.
7. Temporal effectiveness requires no background mutation or scheduler.
8. Persisted `PENDING/ACTIVE` labels are not authoritative domain truth.
9. A pending change may be replaced only before it becomes temporally effective.
10. A temporally effective configuration becomes part of effective domain history; later changes are new changes.
11. Requesting identical effective settings with no meaningful pending difference is a domain no-op.
12. Requesting the effective settings while a non-effective pending change exists cancels that pending change.
13. Initial setup may back-resolve only the Financial Cycle containing initialization.
14. The first revision's `EffectiveFrom` is the start of that initial cycle and establishes the historical floor.
15. No Financial Cycle is invented before the historical floor.
16. Payday Anchor is valid only in `1..31`.
17. Missing anchor dates in short months clamp to the month's final calendar day.
18. Boundary resolution order is anchor → clamp → nominal boundary → weekday adjustment → effective boundary.
19. Working-day adjustment uses weekdays only in V1; no public-holiday calendar is modeled.
20. Weekday adjustment may move an effective boundary into an adjacent calendar month.
21. Short/Extended classification uses the first adjusted/effective regular boundary under the new configuration.
22. Transition `<15` days merges with the following regular cycle into an `EXTENDED` cycle.
23. Transition `>=15` days forms an independent `SHORT` cycle.
24. Exactly 15 transition days is `SHORT`.
25. `CycleType` is assigned by Financial Calendar and is part of authoritative `FinancialCycle` output.
26. `FinancialCycle` is immutable and has no independent identity.
27. Financial Calendar's pure cycle model uses local financial dates; hidden clock/device-timezone access is prohibited.
28. Timezone converts instants to financial dates outside the pure resolver.
29. Changing timezone does not rewrite historical cycle date boundaries.
30. `FinancialCyclePosition` is derived from a cycle and an in-range `AsOfDate`.
31. `TotalCycleDays`, `ElapsedCycleDays`, and `RemainingCycleDays` use inclusive date semantics.
32. On the final cycle date, `RemainingCycleDays = 1`.
33. `ElapsedCycleDays + RemainingCycleDays = TotalCycleDays + 1`.
34. Out-of-range cycle-position dates are rejected, never clamped.
35. Other bounded contexts consume authoritative cycle/position outputs instead of reimplementing their formulas.
36. Primitive Calendar queries use explicit dates; “current” is an application-level composition with Clock + configured timezone.
37. Query contracts do not expose mutable Aggregate internals or generic revision history for cross-context reconstruction.
38. Passage of time alone does not emit a Domain Event.
39. No activation event is required for financial correctness.
40. No-op commands emit no Financial Calendar Domain Event.
41. Persistence normalization/materialization is not a Domain Event.
42. Domain history, Domain Event history, and audit history are distinct and need not correspond one-to-one.
43. Repository operations do not invent business Domain Events.

---

## 19. Decision Register — TD-CAL-001 through TD-CAL-048

| ID | Accepted decision |
|---|---|
| TD-CAL-001 | Financial Cycles are deterministically resolved, not persisted as canonical state. |
| TD-CAL-002 | Store configuration/change facts needed to reconstruct the timeline; cycles remain derived. |
| TD-CAL-003 | Last Pending Configuration Wins while the pending configuration has not become effective. |
| TD-CAL-004 | New configuration is effective immediately after `ProtectedThrough`; `EffectiveFrom = ProtectedThrough + 1 day`. |
| TD-CAL-005 | Effective promotion is temporal/deterministic and requires no background job. |
| TD-CAL-006 | Pending replacement is allowed only before the pending configuration becomes temporally effective. |
| TD-CAL-007 | `PENDING/ACTIVE` are not authoritative persisted lifecycle states; effectiveness is derived from temporal facts. |
| TD-CAL-008 | `FinancialCycle` is an immutable derived Value Object without independent identity. |
| TD-CAL-009 | `CycleType` is authoritative Financial Calendar output and must not be independently inferred by consumers. |
| TD-CAL-010 | Adjusted/effective payday boundary is the authoritative cycle boundary. |
| TD-CAL-011 | Weekday adjustment occurs before Short/Extended transition classification. |
| TD-CAL-012 | Working-day adjustment may move an effective boundary into an adjacent calendar month. |
| TD-CAL-013 | Missing 29/30/31 anchors in short months clamp to the last calendar day. |
| TD-CAL-014 | Boundary resolution order is anchor → clamp → nominal boundary → weekday adjustment → effective boundary. |
| TD-CAL-015 | Financial Calendar models cycle boundaries using local calendar dates, not UTC timestamps, in its pure domain model. |
| TD-CAL-016 | Timezone converts current/evaluation instant to local financial date outside `FinancialCycleResolver`; resolver has no hidden clock/timezone dependency. |
| TD-CAL-017 | Timezone changes do not retroactively rewrite historical Financial Cycle date boundaries. |
| TD-CAL-018 | `FinancialCalendarConfiguration` is the Aggregate Root for configuration-timeline invariants. |
| TD-CAL-019 | Effective `ConfigurationRevision` is an Entity with identity and `EffectiveFrom`; settings are Value Objects. |
| TD-CAL-020 | `PendingConfigurationChange` is a Value Object without independent identity; audit history is separate. |
| TD-CAL-021 | Initial setup may resolve the cycle containing initialization even when its start precedes initialization; initial setup is not a transition. |
| TD-CAL-022 | Requesting settings identical to the effective configuration is a domain no-op when no meaningful pending difference exists. |
| TD-CAL-023 | Requesting the effective settings while a non-effective pending change exists cancels that pending change. |
| TD-CAL-024 | Calendar history begins at the start boundary of the cycle containing `InitializedAt`. |
| TD-CAL-025 | First revision `EffectiveFrom` equals the initial cycle start; `InitializedAt` separately records actual configuration time. |
| TD-CAL-026 | Resolver must not extrapolate before the first revision `EffectiveFrom`; return explicit unavailable/out-of-range result. |
| TD-CAL-027 | V1 mutation surface is limited to `InitializeFinancialCalendar` and `ChangeFinancialCalendarConfiguration`; cycle activation/generation/closing are not commands. |
| TD-CAL-028 | Configuration-change commands evaluate logical temporal state as of `requestedOn`, not stale persisted lifecycle labels. |
| TD-CAL-029 | Change operations distinguish `Scheduled`, `NoChange`, `PendingChangeReplaced`, and `PendingChangeCancelled`. |
| TD-CAL-030 | Distinguish valid domain outcomes, domain rejections, and malformed input/Value Object failures. |
| TD-CAL-031 | Domain Events represent meaningful mutations, not passage of time. |
| TD-CAL-032 | No activation/became-effective event is required for correctness. |
| TD-CAL-033 | Minimal V1 Calendar events distinguish initialization, pending scheduling, replacement, and cancellation. |
| TD-CAL-034 | `NoChange` produces no Financial Calendar Domain Event. |
| TD-CAL-035 | Materializing a temporally effective configuration into a normalized persistence representation is not a Domain Event. |
| TD-CAL-036 | Effective domain history, Domain Event history, and audit history are distinct. |
| TD-CAL-037 | Repository load/normalization/migration/cache/re-persistence must not invent Domain Events. |
| TD-CAL-038 | `ResolveFinancialCycle(requestedDate)` is the authoritative cycle query; consumers must not reconstruct cycles independently. |
| TD-CAL-039 | Resolution returns explicit `Resolved(FinancialCycle)` or a domain outcome such as `CalendarHistoryUnavailable`, not ambiguous `null`. |
| TD-CAL-040 | Primitive Calendar queries accept explicit `requestedDate`/`asOfDate`; current-date convenience use cases compose Clock + timezone outside the resolver. |
| TD-CAL-041 | Query contracts expose purpose-specific DTO/read models, not mutable Aggregate internals or generic revision history. |
| TD-CAL-042 | Financial Calendar owns authoritative `FinancialCyclePosition` derivation separate from `FinancialCycle`. |
| TD-CAL-043 | Cycle-day counts use inclusive semantics; final-day `RemainingCycleDays = 1`. |
| TD-CAL-044 | `FinancialCyclePosition` is derived, not authoritative mutable/persisted daily state. |
| TD-CAL-045 | Consumers use Financial Calendar's elapsed/remaining-day semantics rather than redefining them. |
| TD-CAL-046 | `FinancialCyclePosition.AsOfDate` must be within the supplied cycle; no silent clamping. |
| TD-CAL-047 | Out-of-cycle position calculation is explicit domain rejection and differs from `CalendarHistoryUnavailable`. |
| TD-CAL-048 | Financial Calendar may expose `ResolveFinancialCycleContext(requestedDate)` returning coherent Cycle + Position for the same date. |

---

# Part II — Review & Audit

## 20. Audit Scope

This audit checks the consolidated TD-CAL-001–048 decisions against the frozen Financial Calendar behavior already expressed in the canonical Product Requirements / Financial Engine / Strategic DDD material. The audit does **not** redesign the product and does not silently promote audit suggestions into normative rules.

Audit dimensions:

- contradiction with frozen behavior;
- internal contradiction;
- redundancy/overlap;
- terminology drift;
- hidden implementation coupling;
- missing deterministic semantics that would force an engineer to guess.

---

## 21. Audit Result Summary

**Overall result: PASS WITH CLARIFICATIONS.**

No blocker was found that invalidates the Financial Calendar Tactical model. The central decisions are consistent with the frozen rules: configurable anchor `1..31`, weekday-only adjustment, short-month clamping, preservation of the active cycle, `<15` Extended / `>=15` Short transition strategy, and Financial Calendar as sole temporal authority.

The audit found several places where the candidate should be clarified before freezing, primarily around the interface between local-date Tactical semantics and the Financial Engine's instant-based boundary wording, plus a few intentionally deferred implementation contracts.

---

## 22. Findings

### AUD-CAL-001 — Local-date model vs instant boundary wording — RESOLVED

**Status:** Resolved in this candidate.

`FinancialCycle.StartDate` and `FinancialCycle.EndDate` remain inclusive Local Date values. When an engine or consuming context requires instant/timestamp boundaries, the integration contract is explicitly:

```text
InstantRange =
[start-of-day(StartDate, FinancialTimezone),
 start-of-day(EndDate + 1 day, FinancialTimezone))
```

This preserves the pure local-date domain model while matching the Financial Engine's local-midnight, half-open instant-range semantics.

### AUD-CAL-002 — Timezone integration ownership — RESOLVED

**Status:** Resolved in this candidate.

`FinancialTimezone` is managed by the Application Service / Use Case layer, outside the Financial Calendar Aggregate and `FinancialCycleResolver`. The Application Service reads the system clock, obtains the configured `FinancialTimezone`, converts the instant into `FinancialDate`, then invokes the pure resolver.

### AUD-CAL-003 — `Scheduled` outcome terminology — RESOLVED

**Status:** Resolved in this candidate.

The new-change business outcome is named `Scheduled`, aligned with `PendingCalendarConfigurationScheduled`. This makes clear that the command schedules a future-effective configuration and does not make it immediately effective.

### AUD-CAL-004 — TD-CAL-003 and TD-CAL-006 intentionally overlap

**Severity:** No issue; consolidation opportunity.

TD-CAL-003 establishes “Last Pending Configuration Wins.” TD-CAL-006 narrows its validity to the period before temporal effectiveness. They are not contradictory; TD-CAL-006 is the precision rule for TD-CAL-003.

**Audit recommendation:** In normative prose, express them as one invariant. Keep both IDs in the decision register for traceability.

### AUD-CAL-005 — TD-CAL-005, 007, 031, 032, and 035 form one temporal principle

**Severity:** No issue; consolidation opportunity.

These decisions all protect the same architecture property: time-dependent interpretation must not require scheduler-driven mutation or fabricated events.

**Audit recommendation:** Keep the individual decision IDs but group them under one normative section (“Temporal interpretation without scheduler-driven state mutation”), as this document already does.

### AUD-CAL-006 — Exact persistence representation remains intentionally unresolved

**Severity:** Deferred, not a gap in domain behavior.

The model says a temporally effective pending configuration has effective-revision semantics even if persistence has not been normalized. It intentionally does not decide whether storage physically keeps a transition fact, materializes a revision on later write, or uses another equivalent representation.

**Audit recommendation:** Resolve only during Persistence Design. Add persistence tests proving that reopening the app long after `ProtectedThrough` produces the same effective revision/cycles without requiring a missed scheduler execution.

### AUD-CAL-007 — Revision identity generation is intentionally unspecified

**Severity:** Deferred implementation detail.

`ConfigurationRevision` is correctly modeled as an Entity, but ID generation/type is not specified.

**Audit recommendation:** Leave to implementation architecture/persistence design. No domain behavior currently depends on a particular ID format.

### AUD-CAL-008 — Business financial date vs audit/event timestamp — RESOLVED

**Status:** Resolved in v1.0.

The ambiguous conceptual field `ChangedAt` is replaced by two explicitly typed temporal facts:

- `requestedOn: FinancialDate` — the business financial date used by Financial Calendar rules;
- `occurredAt: Instant` — the audit/event occurrence timestamp.

`occurredAt` MUST NOT implicitly select, alter, or determine a Financial Cycle. If an instant must participate in a Financial Calendar use case, the Application Service / Use Case MUST first convert it through the configured `FinancialTimezone` to an explicit `FinancialDate`.

### AUD-CAL-009 — Initialization historical floor governance — RESOLVED

**Status:** Resolved and reconciled in v1.0.

TD-CAL-021 and TD-CAL-024–026 are formally accepted as product clarifications aligned with the Product Requirements Baseline. They define the initial historical floor and prohibit invented Financial Calendar history before that floor.

### AUD-CAL-010 — Pending replacement/cancellation governance — RESOLVED

**Status:** Resolved and reconciled in v1.0.

TD-CAL-003, TD-CAL-006, TD-CAL-022, and TD-CAL-023 are formally accepted as product clarifications aligned with the Product Requirements Baseline. They define replacement and cancellation behavior for a pending configuration that has not yet become temporally effective.

### AUD-CAL-011 — Cycle position formulas are deterministic and consistent

**Severity:** Pass.

The inclusive formulas preserve the frozen rule that Remaining Cycle Days includes today and equals `1` on the final day. The identity:

```text
ElapsedCycleDays + RemainingCycleDays = TotalCycleDays + 1
```

is internally consistent and suitable for golden/property tests.

### AUD-CAL-012 — Adjustment order is consistent with frozen engine

**Severity:** Pass.

TD-CAL-010–014 match the Financial Engine sequence: nominal anchor uses `min(D, last_day_of_month)`, then weekday adjustment produces the effective anchor. TD-CAL-011 correctly ensures Short/Extended classification uses the adjusted boundary.

### AUD-CAL-013 — No conflict with Event-Driven / Not Event-Sourced architecture

**Severity:** Pass.

The separation of effective domain history, Domain Event history, and audit history is consistent with the Architecture Constitution. The Calendar does not require event replay as source of truth and does not require a transactional message broker/scheduler for correctness.

### AUD-CAL-014 — Query contracts preserve bounded-context ownership

**Severity:** Pass.

`ResolveFinancialCycle`, `ResolveFinancialCycleContext`, and purpose-specific effective/pending configuration reads prevent downstream contexts from reimplementing cycle formulas. Avoiding generic revision-history contracts is consistent with Strategic DDD ownership.

### AUD-CAL-015 — `ResolveFinancialCycleContext` is optional by decision

**Severity:** No issue; implementation/catalog decision remains.

TD-CAL-048 says Financial Calendar **may** expose the coherent Cycle + Position contract. Therefore consumers cannot yet assume this exact query exists until the later Query Catalog/technical architecture selects it.

**Audit recommendation:** Prefer this coherent contract for financial consumers if it reduces mismatched-date errors, but keep its optional status until query catalog review.

---

## 23. Redundancy Map

The following accepted decisions overlap by design and can be represented compactly in normative prose while preserving IDs for traceability:

- **Derived-cycle principle:** TD-CAL-001, 008, 038.
- **Configuration-transition temporal semantics:** TD-CAL-003–007.
- **Boundary resolution:** TD-CAL-010–014.
- **Local-date / timezone purity:** TD-CAL-015–017, 040.
- **Initial history:** TD-CAL-021, 024–026.
- **Command outcomes:** TD-CAL-022–023, 027–030.
- **No scheduler/event-on-time-passage principle:** TD-CAL-005, 007, 031–032, 035–037.
- **Query encapsulation:** TD-CAL-038–041, 048.
- **Cycle position:** TD-CAL-042–047.

No duplicate decision needs deletion; the overlap is useful as a decision trail.

---

## 24. Candidate Test Matrix

These are audit-derived test targets, not new product requirements.

### Boundary resolution

- Anchor `1..31` across all months.
- Leap and non-leap February.
- `NONE`, `PREVIOUS_WORKING_DAY`, `NEXT_WORKING_DAY`.
- Weekend adjustment crossing month/year boundaries.
- Property: consecutive effective regular boundaries are strictly increasing.

### Transition strategy

- Transition 14 days → `EXTENDED`.
- Transition exactly 15 days → `SHORT`.
- Transition 16+ days → `SHORT`.
- Weekend adjustment changes nominal 15-day transition into adjusted `<15` transition.
- Configuration change while current cycle is `REGULAR`, `SHORT`, and `EXTENDED`.

### Pending/effective lifecycle

- Replace pending before effectiveness.
- Cancel pending by returning to effective settings.
- App remains closed across `ProtectedThrough`, then reopens after effectiveness.
- New change after prior pending is temporally effective but physically unnormalized.
- No background scheduler available in any of the above.

### Initial history

- Initialization in middle of cycle back-resolves to cycle start.
- Query on first `EffectiveFrom` succeeds.
- Query one day before historical floor → `CalendarHistoryUnavailable`.

### Cycle position

- First day: `Elapsed = 1`.
- Last day: `Remaining = 1`.
- Identity `Elapsed + Remaining = Total + 1` for every date in cycle.
- Out-of-cycle `AsOfDate` → explicit rejection, no clamp.

### Events

- Initialization emits initialization event.
- Schedule/replace/cancel emit distinct events.
- No-op emits no event.
- Passage of time emits no activation event.
- Persistence normalization emits no business event.

---

## 25. Requirements Governance & Reconciliation Note

All audit items required before freeze have been resolved. No unresolved Financial Calendar item remains in this specification.

The following Tactical DDD rules are formally recorded as approved product clarifications that reconcile and align the Product Requirements Baseline rather than silently overriding it:

1. **Initialization historical floor — TD-CAL-021 and TD-CAL-024–026.** Initial configuration may back-resolve only the Financial Cycle containing `InitializedAt`; the start of that cycle is the Financial Calendar historical floor. Dates before that floor MUST return an explicit unavailable/out-of-range domain result rather than extrapolating invented history.
2. **Pending configuration replacement/cancellation — TD-CAL-003, TD-CAL-006, TD-CAL-022, and TD-CAL-023.** The newest pending configuration replaces an earlier pending configuration only while the earlier change has not become temporally effective. Requesting the currently effective settings cancels a still-non-effective pending change. A configuration that has already become temporally effective is part of effective domain history and cannot be replaced as though it were still pending.

These clarifications are approved as normative product behavior for Financial Calendar v1.0 and are to be treated as reconciled with the Product Requirements Baseline. Future changes require the established product change-control process; downstream architecture or persistence design MUST NOT silently redefine them.

---

## 26. Final Specification Verdict

**Financial Calendar Tactical DDD v1.0: FROZEN SPECIFICATION.**

The Aggregate boundary, temporal model, derived cycle model, configuration transition behavior, commands, queries, events, failure semantics, integration contracts, and consumer contracts are coherent and approved. AUD-CAL-001 through AUD-CAL-010 are resolved/reconciled; later audit observations are either passes or intentionally deferred implementation details that do not block the domain specification.

No additional Financial Calendar feature or rule is required for this v1.0 Tactical DDD baseline. Changes to frozen product behavior require explicit change control. Persistence, Kotlin module layout, concrete repository implementation, and other intentionally deferred implementation details remain subjects for their respective downstream design stages.
