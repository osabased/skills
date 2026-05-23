# Event Topology

Use this file to select the smallest architecture that satisfies the event candidate. Do not start with a broker or branded product.

## Decision Ladder

```text
Can this be a synchronous call without coupling harm?
  yes → direct call/API/module interface
  no  → continue

Is this only inside one process/module boundary?
  yes → local domain event or callback, no broker
  no  → continue

Is there one worker and the producer needs buffering/retry?
  yes → queue/work item
  no  → continue

Do multiple consumers independently react to the same fact?
  yes → pub/sub or event bus
  no  → continue

Do consumers need ordered, replayable history or high-volume append-only processing?
  yes → stream/log
  no  → continue

Does a long-running workflow cross ownership boundaries?
  yes → saga/process manager; choose choreography or orchestration deliberately
  no  → continue

Is the event log the source of truth and replay is a product/operational requirement?
  yes → event sourcing, usually with projections/CQRS
  no  → simpler event notification or event-carried state transfer
```

## Topology Options

### Direct call or request/response

Use when the caller needs an immediate answer, a single owner controls the behavior, or failure must be known synchronously.

Prefer this when async adds only delay and operational risk.

### Local domain events

Use inside one process or bounded context to separate domain behavior from side effects. These are not automatically integration contracts.

Keep local domain events private unless there is an explicit boundary-crossing need.

### Queue / work item

Use when one logical consumer needs buffering, retries, load leveling, background processing, or isolation from producer latency.

Design idempotent handlers because at-least-once delivery is common.

### Pub/sub or event bus

Use when multiple independent consumers need to react to the same fact and the producer should not know every consumer.

The event contract becomes the coupling point. Version it and test consumers.

### Stream / append-only log

Use when ordered history, replay, high-throughput ingestion, event-time processing, or multiple independent consumer positions matter.

Do not choose streams just because they sound more scalable. Streams add partitioning, ordering, retention, schema, and operational concerns.

### Event-carried state transfer

Use when consumers should not call back to the producer for every detail. The event carries enough state for the consumer to update its local view.

This improves autonomy but increases schema/versioning responsibility and data duplication.

### Saga / process manager

Use for long-running business workflows that span multiple ownership or transaction boundaries.

- **Choreography:** each participant reacts to events and emits follow-up events. Good for simple, decentralized flows; risky when flow becomes hard to see.
- **Orchestration:** a coordinator issues commands and tracks progress. Good for complex workflows and explicit recovery; risky when it becomes a god service.

### Projections and read models

Use when read needs differ from write invariants, or when consumers need local query models derived from events.

Define freshness expectations, rebuild process, replay safety, and projection lag visibility.

### Event sourcing

Use only when event history is the system of record and replay/reconstruction has clear value: audit, temporal queries, debugging, compliance, or complex state evolution.

Event sourcing is not required for EDA. It increases design pressure around event immutability, schema migration, snapshots, projections, and privacy deletion.

### CQRS

Use when command-side invariants and query-side read needs are different enough to justify separate models.

CQRS can exist without event sourcing. Event sourcing often needs projections that look CQRS-like.

## Topology Output

```text
TOPOLOGY DECISION
- Chosen topology:
- Alternatives rejected:
- Why this is the smallest sufficient option:
- Producer ownership:
- Consumer ownership:
- Sync/async boundary:
- Consistency model:
- Freshness expectation:
- Operational cost accepted:
```
