# Event Reliability

Most event-driven failures are not caused by publishing syntax. They come from delivery semantics, dual writes, duplicate handling, ordering assumptions, poisoned messages, and invisible lag.

## Reliability Checklist

```text
RELIABILITY DESIGN
- Delivery expectation: best-effort | at-least-once | effectively-once through idempotency | exactly-once within limited system boundary
- Idempotency key:
- Deduplication store/window:
- Ordering key and scope:
- Retry policy:
- Dead-letter policy:
- Poison message handling:
- Outbox needed: yes/no and why
- Inbox needed: yes/no and why
- Replay safe: yes/no and why
- Backpressure behavior:
- Consumer lag visibility:
```

Avoid promising global exactly-once delivery. Prefer practical guarantees: durable publication plus idempotent consumers.

## Dual-Write Risk

If one operation writes business state and another publishes an event, partial failure can make the system inconsistent.

Use a transactional outbox when:

- business state and an event must be committed atomically,
- the producer has a database transaction available,
- losing the event would break downstream behavior,
- publishing directly inside the request path is unreliable or too slow.

```text
TRANSACTIONAL OUTBOX
1. Handle command.
2. Write business state and outbox event in the same transaction.
3. Relay reads unpublished outbox rows.
4. Relay publishes to broker/channel.
5. Relay marks rows published or advances cursor.
6. Consumers remain idempotent because relay may republish.
```

Do not use an outbox for low-value telemetry or local-only callbacks unless durability matters.

## Inbox / Consumer Deduplication

Use an inbox or processed-message table when duplicate handling needs durable memory.

```text
INBOX CHECK
- Receive event.
- Check event id or idempotency key.
- If already processed, acknowledge and stop.
- If new, process and record completion atomically with side effects when possible.
```

Idempotent business operations are better than superficial deduplication. The handler should be safe if the same event arrives twice.

## Ordering

Ordering must have a scope. Global ordering is rarely realistic.

```text
ORDERING REQUIREMENT
- No ordering required.
- Ordered per aggregate/entity ID.
- Ordered per workflow/correlation ID.
- Ordered per partition key.
- Total order required and explicitly justified.
```

If events can arrive out of order, consumers need version checks, sequence numbers, monotonic timestamps, or reconciliation.

## Retries

Retries must avoid amplifying failure.

```text
RETRY POLICY
- Retryable failures:
- Non-retryable failures:
- Max attempts:
- Backoff:
- Jitter:
- Timeout:
- Circuit breaker or pause condition:
```

Do not retry validation failures, unknown schema versions, authorization failures, or deterministic business rejections forever.

## Dead Letters and Poison Messages

Dead-letter queues are not a solution by themselves. Define who looks at them and how recovery works.

```text
DLQ POLICY
- Goes to DLQ when:
- Alert threshold:
- Owner:
- Triage data included:
- Replay path:
- Drop path:
- Privacy constraints:
```

## Replay

Replay changes the meaning of side effects. A replay-safe handler can rebuild projections without resending emails, charging cards, or calling irreversible external APIs.

Classify consumers:

```text
REPLAY CLASS
- Projection-only: safe to replay.
- Idempotent side effect: replay safe with idempotency keys.
- Irreversible side effect: not replay safe; isolate behind command or manual gate.
```

## Observability

Every non-trivial event flow needs correlation.

Track:

- event ID,
- correlation ID,
- causation ID,
- producer and consumer names,
- schema/type/version,
- publish latency,
- consumer lag,
- retry count,
- dead-letter count,
- handler duration,
- projection staleness.

A system that cannot show where an event is stuck is not production-ready.
