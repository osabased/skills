# Event-Driven Architecture Anti-Patterns

Check this file before recommending a broker, async rewrite, saga, CQRS, or event sourcing.

## Decoupling Theater

Symptom: The plan says events will "decouple services" but does not name concrete async, fan-out, autonomy, throughput, durability, replay, or integration needs.

Correction: Treat the event contract, delivery semantics, schema, and operations as the new coupling. Use direct calls unless the tradeoff pays.

## Broker-First Design

Symptom: The architecture starts with Kafka, RabbitMQ, NATS, EventBridge, SNS/SQS, Service Bus, Redis Streams, or another product before the workflow is understood.

Correction: Choose topology first. Choose product later with `source-driven-development` and project constraints.

## Event Sourcing by Default

Symptom: Every domain event becomes a persisted event log and source of truth.

Correction: Event sourcing is an advanced persistence architecture for replay/audit/temporal reconstruction needs. Most EDA uses ordinary state plus integration events.

## CRUD Over Events

Symptom: Events are named `EntityCreated`, `EntityUpdated`, and `EntityDeleted` without business meaning.

Correction: Prefer domain facts such as `InvoiceIssued` or `AccessRevoked`. Use CRUD events only for low-level data sync where that is honestly the contract.

## Command as Event

Symptom: Event names are imperative: `ProcessPayment`, `SendEmail`, `UpdateSearchIndex`.

Correction: Commands ask for work; events state facts. Rename or redesign.

## Hidden Synchronous Dependency

Symptom: Producer publishes an event, but consumers immediately call back to the producer for required state, creating latency and availability coupling.

Correction: Either use direct request/response or event-carried state transfer with explicit schema and freshness semantics.

## Dual-Write Hand Wave

Symptom: The plan writes to the database and publishes an event with no transactional story.

Correction: Use a transactional outbox or prove that losing/duplicating the event is acceptable.

## Non-Idempotent Consumer

Symptom: A duplicate event can send two emails, charge twice, reserve twice, or corrupt a projection.

Correction: Add idempotency keys, inbox/deduplication, or idempotent domain operations before shipping.

## Ordering Illusion

Symptom: The design assumes events arrive in the order they were emitted across the whole system.

Correction: Define ordering scope. Use partition keys, sequence numbers, version checks, or reconciliation.

## Infinite Retry Loop

Symptom: All failures retry forever.

Correction: Classify retryable vs non-retryable failures, add bounded backoff, DLQ ownership, alerting, and recovery paths.

## Dead-Letter Graveyard

Symptom: Failed events are moved to a DLQ but nobody owns triage or replay.

Correction: Define owner, alert threshold, triage data, replay/drop process, and privacy constraints.

## Schema Drift

Symptom: Producers change payloads without consumer compatibility checks.

Correction: Treat events as APIs. Add schema versioning, producer tests, consumer contract tests, and migration windows.

## Choreography Spaghetti

Symptom: Many services react to each other through events and no one can explain the workflow end to end.

Correction: Introduce an explicit process manager/orchestrator or document the saga flow with ownership and stop conditions.

## Replay Side Effects

Symptom: Replaying events resends emails, charges cards, or triggers external calls.

Correction: Separate projection rebuilds from irreversible side effects. Gate side effects behind commands and idempotency.

## Event Bus as Global Shared Database

Symptom: Consumers depend on every field from every event, and producers cannot change anything safely.

Correction: Define published language, ownership, minimal payloads, compatibility rules, and consumer-specific projections when needed.
