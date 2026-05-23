# Lightweight Event Storming

Use this when workflow order, state transitions, responsibilities, or hidden business rules are unclear.

This is not a workshop simulator. It is a fast modeling pass that turns messy behavior into commands, events, policies, actors, read models, and questions.

## Inputs

Collect whatever exists:

- User story or feature request.
- Support tickets or bug reports.
- Existing tests.
- API/routes/controllers.
- Database tables or schemas.
- Logs or audit trails.
- Screens and user-facing copy.
- Domain expert notes.

## Pass 1: Domain Events

Start with things that happened. Use past tense.

```text
DOMAIN EVENTS
1. [Thing]Requested
2. [Thing]Validated
3. [Thing]Approved
4. [Thing]Rejected
5. [Thing]Published
```

Good event names are domain facts, not technical callbacks.

Bad: `FormSubmitted`, `HandlerCalled`, `RecordUpdated`.
Good: `ApplicationSubmitted`, `PaymentAuthorized`, `ShipmentScheduled`.

## Pass 2: Commands

For each event, ask what request caused it.

```text
COMMAND
- Command:
- Actor/system:
- Produces event(s):
- Preconditions:
- Rejection reasons:
```

Commands are attempts. Events are facts.

## Pass 3: Policies

Find rules that react to events and trigger commands or decisions.

```text
POLICY
- When:
- If:
- Then:
- Owner/source:
- Failure mode:
```

## Pass 4: Aggregates or Consistency Boundaries

Group commands/events around invariants that must be protected atomically.

```text
CONSISTENCY BOUNDARY
- Boundary name:
- Commands handled:
- Events emitted:
- Invariants protected:
- Data needed:
- External data referenced only:
```

## Pass 5: Read Models

Identify views, reports, projections, search screens, dashboards, or API responses that do not need to enforce invariants.

```text
READ MODEL
- Name:
- Users:
- Questions answered:
- Source events/entities:
- Freshness requirement:
```

## Pass 6: Hotspots and Questions

Mark uncertainty instead of smoothing over it.

```text
HOTSPOTS
- Ambiguous term:
- Missing rule:
- Conflicting source:
- External dependency:
- Ordering/concurrency risk:
```

## Output

```text
EVENT STORMING SUMMARY
TIMELINE
- Command -> Event -> Policy -> Command/Event

RULES DISCOVERED
- ...

INVARIANTS
- ...

READ MODELS
- ...

OPEN QUESTIONS
- ...

NEXT ARTIFACT UPDATES
- CONTEXT.md:
- docs/domain-model.md:
- docs/event-map.md or event-driven-architecture handoff, if events cross runtime/integration boundaries:
- tests/examples:
```
