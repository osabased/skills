# Event Contracts

An event contract is the public interface of an event-driven system. Treat it with the same care as an API.

## Naming

Events are facts that already happened. Use past tense and domain language.

Good:

```text
OrderPlaced
PaymentAuthorized
InvoiceIssued
ShipmentScheduled
AccessRevoked
```

Bad:

```text
ProcessOrder
SendEmail
UpdateDatabase
HandlerCalled
RecordChanged
```

Do not leak implementation details into event names. A consumer should understand the business/system fact without knowing the producer's internals.

## Event Kind

```text
EVENT KIND
- Domain event: internal modeling fact within one boundary
- Integration event: public fact crossing a boundary
- Notification event: announces fact; consumers may fetch details
- Event-carried state transfer: includes enough state for local consumer updates
- Event-sourced event: immutable source-of-truth state transition
```

Pick one primary kind. Mixing kinds creates unclear contracts.

## Contract Template

```text
EVENT CONTRACT
- Name:
- Kind:
- Owner/producer:
- Meaning:
- Emitted when:
- Not emitted when:
- Consumers:
- Payload fields:
- Required metadata:
- Idempotency key:
- Ordering key, if any:
- Version:
- Compatibility rule:
- Retention/replay expectation:
- Privacy/security classification:
- Example event:
```

## Envelope

For cross-boundary or long-lived events, prefer a stable envelope.

```json
{
  "id": "event-unique-id",
  "source": "producer-or-context-name",
  "type": "order.placed.v1",
  "subject": "order/123",
  "time": "2026-01-01T12:00:00Z",
  "datacontenttype": "application/json",
  "dataschema": "schema-uri-or-name",
  "correlation_id": "request-or-workflow-id",
  "causation_id": "command-or-prior-event-id",
  "data": {}
}
```

Use a CloudEvents-style shape when interoperability matters. Do not force this exact envelope on tiny local-only events if it adds no value.

## Payload Design

Choose payload shape based on consumer autonomy.

### Notification payload

```json
{
  "order_id": "ord_123"
}
```

Use when consumers can safely fetch current state and do not need historical details.

Risk: callback dependency, thundering herds, consumers seeing newer state than the event's moment.

### Event-carried state payload

```json
{
  "order_id": "ord_123",
  "customer_id": "cus_456",
  "status": "placed",
  "total": { "amount": "42.00", "currency": "USD" },
  "placed_at": "2026-01-01T12:00:00Z"
}
```

Use when consumers need a local view without calling back.

Risk: schema bloat, duplicated data, privacy exposure, evolution burden.

## Versioning Rules

Prefer additive changes. A new optional field is usually compatible. Removing, renaming, changing type, changing meaning, or changing emission semantics is breaking.

```text
SCHEMA EVOLUTION CHECK
[ ] Are old consumers safe if they ignore new fields?
[ ] Are new consumers safe with old events?
[ ] Is the event meaning unchanged?
[ ] Are required fields still present?
[ ] Are enum changes handled?
[ ] Is replay of old events still valid?
```

If not, publish a new event type/version and run both during migration.

## Ownership

Each event has one owning producer. Consumers may request changes, but the owner controls meaning and compatibility.

Do not let consumers infer private producer state from undocumented fields. If a field is consumed, it is part of the contract.

## Contract Tests

At minimum, test:

- producer emits the event with valid schema for the triggering behavior,
- consumer accepts current and previous compatible versions,
- duplicate events are safe,
- unknown optional fields are ignored,
- missing required fields fail clearly,
- replay does not cause unsafe side effects.
