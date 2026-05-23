# Domain Artifacts

This skill may create or update domain documentation directly when the model is supported by evidence. Prefer small, durable updates over large speculative docs.

## `CONTEXT.md`

Use for stable vocabulary that future agents should reuse.

```md
# Context

## Domain Language

### [Term]

- **Meaning:** ...
- **Context:** ...
- **Not the same as:** ...
- **Source/evidence:** ...
- **Status:** stable | provisional | disputed
```

Rules:

- Add stable terms when source evidence is strong enough to guide future implementation, tests, or docs.
- If stability is uncertain, propose the term in the report instead of writing it to `CONTEXT.md`.
- Mark disputed terms as provisional or disputed.
- Do not redefine terms already settled by an ADR unless explicitly reopening the decision.

## `docs/domain-map.md`

Use for strategic boundaries.

```md
# Domain Map

## Subdomains

| Subdomain | Type | Why it matters | Evidence |
|---|---|---|---|
| ... | core/supporting/generic | ... | ... |

## Bounded Contexts

### [Context Name]

- **Purpose:** ...
- **Owns:** ...
- **Does not own:** ...
- **Local language:** ...
- **Public contract:** ...

## Context Relationships

| Upstream | Downstream | Relationship | Translation | Risk |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |
```

## `docs/domain-model.md`

Use for behavior and tactical model decisions.

```md
# Domain Model

## Commands

| Command | Actor | Preconditions | Produces | Rejection reasons |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## Events

| Event | Meaning | Payload | Consumers | Ordering/idempotency |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## Policies

| Policy | Decision | Inputs | Output | Rule source |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## Invariants

| Invariant | Protected by | Acceptance example | Test seam |
|---|---|---|---|
| ... | ... | ... | ... |

## Aggregate Candidates

### [Aggregate Root]

- **Invariants:** ...
- **Commands:** ...
- **Events:** ...
- **Inside boundary:** ...
- **Referenced by identity:** ...
- **Concurrency risk:** ...
```

## ADRs

Create an ADR only when the decision is durable and future agents are likely to revisit it.

```md
# ADR-[number]: [Decision]

## Status

Accepted | Proposed | Superseded

## Context

What domain ambiguity or boundary pressure forced this decision?

## Decision

What model, boundary, term, or relationship is chosen?

## Consequences

- Positive:
- Negative:
- Follow-up:
```

Do not create ADRs for temporary priority, obvious naming, or implementation details that can be discovered from code.
