# Strategic Design

Use strategic DDD to decide where language and ownership change. This is the default branch of the skill.

## Subdomains

Classify domain areas by business importance, not by code folder.

```text
SUBDOMAIN
- Name:
- Type: core | supporting | generic
- Why it matters:
- Main actors:
- Differentiating rules:
- Current code/docs evidence:
```

### Core domain

The part that makes the product strategically valuable. It deserves the most careful language, modeling, tests, and architecture.

Signals:

- The user's product would lose its reason to exist if this behavior were generic.
- Rules are nuanced, changing, or business-specific.
- Mistakes here are expensive or user-visible.

### Supporting subdomain

Important, but not the main differentiator. It should be clear and reliable, but usually needs less modeling depth than the core domain.

### Generic subdomain

Necessary commodity capability: authentication, billing integration, logging, email transport, storage plumbing, analytics export, etc. Prefer existing tools and straightforward integration unless the product's domain makes it special.

## Bounded Contexts

A bounded context is where a model's terms have one precise meaning. Boundaries are justified by language, rules, lifecycle, ownership, or consistency needs.

```text
BOUNDED CONTEXT CANDIDATE
- Name:
- Purpose:
- Terms with local meaning:
- Owns:
- Does not own:
- Upstream/downstream contexts:
- Public contract:
- Internal rules:
- Evidence:
```

Good boundary signals:

- The same word means different things in different workflows.
- Two areas change for different reasons.
- One area needs strong invariants while another only needs a projection/read model.
- Integration with an external model would pollute local language without translation.
- Tests or feature docs naturally describe different capabilities.

Weak boundary signals:

- Folder names happen to be different.
- A framework generated separate layers.
- A class or table exists.
- Someone wants symmetry or pattern completeness.

## Context Mapping

Map relationships between contexts before proposing code movement.

```text
CONTEXT MAP RELATIONSHIP
- Upstream context:
- Downstream context:
- Relationship: customer-supplier | conformist | anti-corruption layer | shared kernel | published language | open-host service | separate ways
- Contract:
- Translation needed:
- Risk:
```

### Relationship guide

- **Customer-supplier**: downstream needs influence upstream; coordinate changes deliberately.
- **Conformist**: downstream accepts upstream's model because negotiation is not realistic.
- **Anti-corruption layer**: downstream protects its model by translating upstream concepts.
- **Shared kernel**: two contexts share a small model by agreement; keep it tiny and versioned.
- **Published language**: upstream exposes a stable public contract that downstreams can rely on.
- **Open-host service**: upstream provides an integration surface for many consumers.
- **Separate ways**: contexts do not integrate because integration cost exceeds value.

## Boundary Decision Checklist

Before recommending a boundary, answer:

```text
[ ] What term/rule/lifecycle differs across the proposed boundary?
[ ] Who owns the source of truth?
[ ] What invariant must never be split accidentally?
[ ] What data crosses the boundary?
[ ] Is translation required, or is shared language safe?
[ ] What test or example proves the boundary is real?
[ ] What would get worse if these stayed together?
[ ] What would get worse if these were separated?
```

If the checklist is weak, mark the boundary as speculative instead of architectural fact.
