# Event Architecture Artifacts

Create or update artifacts only when they will guide future implementation, review, or debugging. Prefer small durable files over speculative architecture documents.

## `docs/event-map.md`

Use for system-level event flows.

````md
# Event Map

## Scope

- Feature/workflow:
- In scope:
- Out of scope:
- Consistency model:

## Event Flow

| Producer | Event | Kind | Consumers | Consumer action | Freshness | Failure handling |
|---|---|---|---|---|---|---|
| ... | ... | domain/integration/notification/state-transfer/sourced | ... | ... | ... | ... |

## Topology

- Chosen topology:
- Alternatives rejected:
- Why this is the smallest sufficient option:

## Workflow Notes

```text
Command -> Producer -> Event -> Consumer/Policy -> Command/Event/Projection
```

## Open Questions

| Question | Why it matters | Owner | Blocking? |
|---|---|---|---|
| ... | ... | ... | yes/no |
````

## `docs/event-contracts.md`

Use when events cross boundaries or have multiple consumers.

````md
# Event Contracts

## [event.type.v1]

- **Name:**
- **Kind:**
- **Owner/producer:**
- **Meaning:**
- **Emitted when:**
- **Not emitted when:**
- **Consumers:**
- **Compatibility rule:**
- **Idempotency key:**
- **Ordering key:**
- **Replay expectation:**

### Schema

```json
{
  "id": "...",
  "source": "...",
  "type": "...",
  "time": "...",
  "data": {}
}
```

### Examples

```json
{}
```
````

## `docs/event-reliability.md`

Use when the flow has retries, outbox/inbox, replay, DLQ, ordering, or operational risk.

````md
# Event Reliability

## Delivery Guarantees

| Event | Delivery expectation | Idempotency | Ordering | Retry | DLQ | Replay safe |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... |

## Outbox/Inbox

- Producer outbox:
- Consumer inbox:
- Transaction boundary:
- Relay behavior:

## Observability

- Logs:
- Metrics:
- Traces:
- Alerts:
- Dashboards:
````

## ADRs

Create an ADR when the event topology or reliability decision is durable and future agents are likely to revisit it.

````md
# ADR-[number]: [Event Architecture Decision]

## Status

Accepted | Proposed | Superseded

## Context

What async, integration, consistency, or reliability pressure forced this decision?

## Decision

What topology, contract, broker-agnostic pattern, or reliability mechanism is chosen?

## Alternatives Considered

- Direct call:
- Queue:
- Pub/sub:
- Stream:
- Saga/process manager:
- Event sourcing/CQRS:

## Consequences

- Positive:
- Negative:
- Operational requirements:
- Testing requirements:
````

## Artifact Rules

- Do not create event docs for private local callbacks.
- Do not document speculative consumers as if they exist.
- Mark unknown consumers, ordering requirements, and retry policies as open questions.
- Keep contracts close to code if the project already has schema files or API specs.
- Use `source-driven-development` before writing vendor-specific configuration.
