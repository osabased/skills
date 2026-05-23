# Tactical Design

Use tactical DDD only after the domain slice, language, and boundaries are clear enough. Tactical patterns are tools for protecting business rules, not a mandatory object model.

## Pattern Selection Rule

Pick the smallest construct that protects the rule.

```text
RULE OR CONCEPT
- What must stay true?
- Where can it be violated?
- Who is allowed to change it?
- Does it need identity over time?
- Does it need atomic consistency with related changes?
- Is persistence an implementation detail or part of the contract?
```

## Value Objects

Use for concepts defined by their attributes rather than identity.

Good candidates:

- Money, DateRange, Quantity, EmailAddress, Version, Score, Measurement, Slug, Percentage.
- Any concept with validation, normalization, comparison, arithmetic, or formatting rules.

```text
VALUE OBJECT
- Name:
- Attributes:
- Validation rules:
- Equality rule:
- Operations:
- Invalid states prevented:
```

## Entities

Use when identity and lifecycle matter across changes.

```text
ENTITY
- Name:
- Identity:
- Lifecycle states:
- Allowed transitions:
- Rules tied to identity:
- External references:
```

Do not make every noun an entity. If identity does not matter, prefer a value object or plain data.

## Aggregates

Use aggregates when a cluster of objects must preserve invariants under one consistency boundary.

```text
AGGREGATE CANDIDATE
- Root:
- Invariant(s):
- Objects inside boundary:
- Objects referenced by ID only:
- Commands handled:
- Events emitted:
- Transaction boundary:
- Concurrency risk:
```

Aggregate rules:

- One aggregate protects one consistency boundary.
- External code talks to the aggregate root, not internals.
- Keep aggregates small enough to load, reason about, and test.
- Reference other aggregates by identity unless strong consistency truly requires otherwise.
- Do not create aggregates just because an ORM relationship exists.

## Domain Services and Policies

Use a domain service or policy when behavior is real domain logic but does not naturally belong to one entity/value object.

```text
POLICY / DOMAIN SERVICE
- Decision made:
- Inputs:
- Output:
- Rule source:
- Pure or stateful:
- Why it does not belong inside an entity/value object:
```

Prefer pure policies for rules like eligibility, pricing decisions, ranking, scheduling, authorization decisions, and routing decisions when they are domain-specific.

## Repositories

Use repositories only when the domain model needs a collection-like abstraction for aggregate persistence or lookup.

```text
REPOSITORY
- Aggregate root:
- Lookup methods required by domain use cases:
- Save semantics:
- Consistency/concurrency expectations:
- Query use cases intentionally excluded:
```

Avoid repositories that are just generic table gateways. Read-heavy screens may be better served by query/read-model adapters rather than forcing everything through aggregates.

## Domain Events

Use domain events for facts that already happened and matter to other parts of the system.

```text
DOMAIN EVENT
- Name, past tense:
- When emitted:
- Meaning:
- Payload:
- Consumers:
- Ordering/idempotency requirements:
```

Do not use events to hide direct control flow inside one simple module. Use them when decoupling, auditability, or workflow progression is worth the operational cost. Route to `event-driven-architecture` before turning domain events into public integration events, async workflows, broker topics, streams, or replayable contracts.

## Factories

Use factories when constructing a valid domain object requires named rules, non-trivial defaults, generated identity, or multi-step normalization.

```text
FACTORY
- Creates:
- Inputs:
- Defaults/generated values:
- Rules enforced:
- Failure modes:
```

## Implementation Handoff

Before coding, produce:

```text
TACTICAL IMPLEMENTATION HANDOFF
- Public behavior:
- Domain terms used in names:
- Invariants to test:
- Chosen constructs:
- Persistence/integration boundaries:
- First red-green behavior:
```

Then route to `shift-left-testing` or `test-driven-development`.
