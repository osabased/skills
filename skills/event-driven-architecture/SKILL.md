---
name: event-driven-architecture
description: Guides agents through event-driven architecture decisions, including signal classification, async justification, topology selection, event contracts, reliability, observability, and verification. Use when async workflows, pub/sub, queues, streams, event buses, integration events, outbox/inbox flows, sagas, projections, replay, CQRS, or event sourcing are proposed or being reviewed. Use when a domain event may need to become a runtime or integration event.
---

# Event-Driven Architecture

## Overview

Use events to communicate facts that already happened across meaningful boundaries. Do not use events as a default architecture style, a vague decoupling mechanism, or a way to avoid clear ownership.

Event-driven architecture moves complexity from direct calls into contracts, delivery timing, ordering, duplication, replay, schema evolution, and operations. The job of this skill is to decide whether events are justified, choose the smallest viable topology, design the event contract, plan reliability before implementation, and require proof that the flow works under failure.

`SKILL.md` is the workflow entry point. Load supporting references only when the current step needs more detail.

## When to Use

- A feature crosses service, module, bounded-context, workflow, or integration boundaries
- Multiple consumers need to react to the same fact
- A workflow needs async progress, buffering, fan-out, retry, independent deployment, throughput isolation, auditability, replay, or eventual consistency
- The design mentions pub/sub, queues, streams, brokers, event buses, integration events, outbox, inbox, sagas, process managers, projections, event contracts, CQRS, or event sourcing
- A `domain-driven-design` pass produced domain events and the next question is whether any should become runtime/integration events
- There is risk around duplicate delivery, ordering, retries, schema evolution, poison messages, dead letters, replay safety, or observability

**When NOT to use:** Don't introduce runtime events for simple CRUD, one-consumer control flow, in-process callbacks, vague "decoupling," or domain modeling that has not clarified language, invariants, commands, policies, and boundaries. Use `domain-driven-design` first when the domain is still unclear.

## The Event-Driven Architecture Workflow

```
1. CLASSIFY  → Is this actually an event?
2. JUSTIFY   → Prove async events are worth the operational cost
3. MAP       → Identify producers, consumers, boundaries, and side effects
4. SELECT    → Choose the lightest viable topology
5. CONTRACT  → Define event names, payloads, metadata, versions, and ownership
6. RELIABLY  → Design idempotency, ordering, retries, DLQs, outbox/inbox, replay
7. OBSERVE   → Add traces, logs, metrics, lag, recovery, and ownership
8. VERIFY    → Prove the event flow works, including failure paths
```

### Step 1: Classify the Signal

Before designing anything, decide what kind of message this is:

| Candidate | Meaning | Default Direction |
|---|---|---|
| **Command** | "Please do this" | Direct call, command handler, queue only if async execution is required |
| **Query** | "Please return this data" | Synchronous read/API, projection only if read shape or scale justifies it |
| **Domain event** | "Something meaningful happened inside the domain" | Keep internal until it must cross a boundary |
| **Integration event** | "Something happened that external consumers may rely on" | Public contract with versioning and compatibility rules |
| **Local callback** | "Notify code in the same process" | Simple function call or local event mechanism |

Use this template:

```text
EVENT CANDIDATE
- Fact that already happened:
- Producer:
- Owner:
- Consumers:
- Boundary crossed:
- Why direct call may be insufficient:
- Freshness requirement: immediate | seconds | minutes | eventual | replayable
- Failure cost:
- Duplicate delivery cost:
- Ordering requirement:
```

Exit early if the "event" is really a command, query, direct call, or local one-consumer flow.

### Step 2: Justify EDA

Name the specific reason events are better than a direct call:

- fan-out to multiple consumers
- buffering or backpressure
- retry after transient failure
- independent deployment or autonomy
- throughput isolation
- eventual consistency is acceptable and useful
- audit trail or replay is required
- integration with external systems
- long-running workflow coordination
- read-model/projection maintenance

If the only reason is "decoupling," reject the design. Events do not remove coupling; they move it into contracts, delivery guarantees, timing assumptions, replay behavior, and operations.

### Step 3: Map the Flow

Create a minimal event map before choosing infrastructure:

```text
Producer/Owner ──publishes──> EventName.vN ──consumed by──> Consumer
      │                            │                         │
      └─ transaction boundary       └─ contract owner          └─ side effect
```

For each event, record:

- producer and owner
- domain boundary or integration boundary crossed
- current consumers and expected future consumers
- side effects each consumer performs
- whether consumers need callback reads
- whether consumers can tolerate delay
- failure owner for publication and consumption

Do not let private domain events leak into public integration contracts without deliberately translating them.

### Step 4: Choose the Topology

Start with the simplest topology that satisfies the event map.

| Need | Prefer | Avoid Until Proven Necessary |
|---|---|---|
| Notify code inside one process | Function call or local event | Broker |
| Run one async worker later | Queue | Pub/sub or stream |
| Let many consumers react independently | Pub/sub/event bus | Point-to-point chains |
| Preserve ordered log, replay, or high-volume stream processing | Stream/log | Event sourcing |
| Coordinate a long-running multi-step workflow | Saga/process manager | Distributed transaction |
| Build read-optimized views | Projection/CQRS read model | Full CQRS architecture everywhere |
| Store state as event history | Event sourcing | Event sourcing by default |

Load [TOPOLOGY.md](TOPOLOGY.md) when the topology choice is non-obvious, high-risk, or involves streams, sagas, CQRS, event sourcing, or replay.

### Step 5: Design the Event Contract

A boundary-crossing event must have a contract before implementation.

```text
EVENT CONTRACT
- Name:
- Kind: domain | integration | notification | event-carried state transfer | event-sourced
- Meaning:
- Owner:
- Producer:
- Emitted when:
- Not emitted when:
- Consumers:
- Version:
- Payload:
- Required metadata:
- Idempotency key:
- Ordering key, if any:
- Compatibility rule:
- Retention/replay rule:
- Privacy/security classification:
- Consumer expectations:
- Example event:
```

Contract rules:

- Name events as facts, not commands: `InvoicePaid`, not `PayInvoice`.
- Keep ownership explicit: consumers do not own producer data.
- Include enough data for consumers to act deliberately; avoid accidental database dumps.
- Decide whether callback reads are allowed or forbidden.
- Version for compatibility before multiple consumers depend on the event.
- Treat public integration events as long-lived APIs.

Load [EVENT-CONTRACTS.md](EVENT-CONTRACTS.md) for public contracts, schema evolution, CloudEvents-style metadata, compatibility rules, examples, or multi-consumer flows.

### Step 6: Design Reliability Before Coding

Every durable event flow needs a reliability stance:

| Concern | Required Decision |
|---|---|
| Transaction boundary | How the state change and event publication stay consistent |
| Delivery expectation | at-most-once, at-least-once, effectively-once, or best-effort |
| Idempotency | How consumers handle duplicates |
| Ordering | Ordering key, ordering scope, and what happens out of order |
| Retries | Which failures retry, how often, and when they stop |
| Poison messages | Dead-letter handling and ownership |
| Replay | Whether replay is safe, supported, forbidden, or migration-only |
| Backpressure | What happens when consumers fall behind |
| Inbox/outbox | Whether durable publication/consumption tracking is required |

Database write plus event publication must address the dual-write problem before implementation. Load [RELIABILITY.md](RELIABILITY.md) when persistence, brokers, retries, replay, outbox/inbox, or cross-boundary delivery are involved.

### Step 7: Define Observability and Operations

Async failures are easy to hide. Define evidence before launch:

- correlation ID from producer to every consumer
- structured logs for publish, consume, retry, skip, DLQ, and replay
- metrics for publish count, consume count, error rate, retry count, DLQ depth, lag, and processing latency
- traces across producer, broker/transport, and consumers when supported
- dashboard or command for checking event health
- owner for failed messages
- manual recovery path for poison messages and stuck consumers

An operator should be able to answer: what was published, who consumed it, what failed, whether it will retry, and how to recover.

### Step 8: Verify and Hand Off

For architecture-only work, produce a decision report. For implementation work, route to `shift-left-testing` for contract, consumer, replay, integration, and failure-injection gates. Route to `source-driven-development` before using a specific broker, framework, cloud service, SDK, or schema registry.

Use this handoff:

```text
EVENT-DRIVEN ARCHITECTURE DECISION
- Decision: use events | do not use events | use local events only | defer
- Reason:
- Rejected alternatives:
- Event map:
- Topology:
- Contracts:
- Reliability:
- Observability:
- Verification plan:
- Artifact updates:
- Open questions:
- Stop condition:
```

## EDA Defaults

Use conservative defaults unless the system proves it needs more:

```
Default to direct calls before async events.
Default to local events before brokers.
Default to queues before pub/sub when there is only one consumer.
Default to pub/sub before streams when replay and ordered logs are not required.
Default to projections before broad CQRS.
Default away from event sourcing unless event history is the source of truth.
Default to at-least-once assumptions for durable messaging.
Default to idempotent consumers.
Default to explicit contracts for boundary-crossing events.
```

## Common Anti-Patterns

#### Broker-First Design

```text
BAD: "This crosses a module, so add an event bus."
GOOD: "This has three independent consumers and can tolerate eventual consistency, so use pub/sub."
```

#### Command Disguised as Event

```text
BAD: SendInvoiceEmail
GOOD: InvoicePaid → BillingEmailConsumer sends email
```

#### Event Sourcing Theater

```text
BAD: "We use events, so the database should be reconstructed from them."
GOOD: "We need a durable audit log and temporal reconstruction, so event sourcing may be justified."
```

#### Hidden Callback Coupling

```text
BAD: Event contains only userId; every consumer calls the producer immediately.
GOOD: Event carries the stable facts consumers need, or callback reads are explicitly designed and rate-limited.
```

## See Also

Load supporting files only when needed:

- [TOPOLOGY.md](TOPOLOGY.md) — direct calls, local events, queues, pub/sub, streams, event buses, sagas, projections, CQRS, event sourcing
- [EVENT-CONTRACTS.md](EVENT-CONTRACTS.md) — names, payloads, metadata, ownership, schema evolution, examples
- [RELIABILITY.md](RELIABILITY.md) — outbox/inbox, idempotency, ordering, retries, DLQs, replay, backpressure
- [ARTIFACTS.md](ARTIFACTS.md) — event maps, contract docs, reliability docs, ADR templates
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md) — broker-first design, over-async, fake decoupling, event-sourcing theater

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Events decouple everything" | Events shift coupling into contracts, timing, delivery, replay, schema evolution, and operations. Name the coupling. |
| "We'll add retries and idempotency later" | Reliability affects contracts and side effects. Design it before consumers depend on the flow. |
| "A broker makes this scalable" | Brokers add buffering and routing; they do not remove capacity, partitioning, backpressure, or recovery work. |
| "Events mean event sourcing" | Event sourcing makes events the source of truth. Most EDA does not need that. |
| "Consumers can just call back for details" | Callback reads create hidden coupling, timing races, load spikes, and extra failure modes. Decide deliberately. |
| "Ordering is guaranteed" | Ordering is scoped and costly. State the ordering key, boundary, and fallback. |
| "At-least-once is fine" | At-least-once requires idempotent consumers and duplicate-safe side effects. |
| "We'll monitor after launch" | Async failures are invisible without correlation, lag, retry, DLQ, and replay evidence. |
| "This is future-proof" | Unused event infrastructure is architectural debt. Prove the current need. |

## Red Flags

- Event name is imperative, vague, or describes an action instead of a fact
- Producer, owner, or consumers are unknown
- Events are chosen only to avoid a direct dependency
- Private domain events are exposed as public integration contracts
- Payload is ID-only with required callback reads, or a database dump with unclear ownership
- No idempotency, retry, DLQ, replay, ordering, or backpressure stance
- Design assumes exactly-once delivery, global ordering, or safe replay without evidence
- Event sourcing, CQRS, sagas, or streams are selected before simpler options are rejected
- Consumers have side effects but no duplicate-handling strategy
- No trace from publication to consumer effects
- Tests cover only the happy path

## Verification

After any event-driven architecture decision:

- [ ] The event candidate was classified as command, query, domain event, integration event, or local callback
- [ ] EDA justification names a concrete benefit beyond "decoupling"
- [ ] Rejected alternatives are documented
- [ ] Event map lists producers, owners, events, consumers, boundaries, and side effects
- [ ] Topology is the lightest option that satisfies the requirements
- [ ] Event contracts include name, meaning, owner, payload, metadata, versioning, compatibility, and example events
- [ ] Reliability plan covers transaction boundary, idempotency, retries, ordering, DLQ, replay, backpressure, and outbox/inbox where needed
- [ ] Observability plan covers correlation, logs, metrics, traces, lag, retries, DLQ, and recovery ownership
- [ ] Contract/consumer tests are planned or implemented
- [ ] Failure-path tests cover duplicate delivery, retry, poison message, DLQ, and recovery where applicable
- [ ] Replay or migration tests exist when historical events may be reprocessed
- [ ] Exact commands, logs, test output, or CI evidence are attached when implementation exists

If evidence cannot be run, state what is unverified and give the exact command or manual check needed next.
