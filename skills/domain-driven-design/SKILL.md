---
name: domain-driven-design
description: Guides agents through Domain-Driven Design for complex business domains, including ubiquitous language, subdomains, bounded contexts, workflows, invariants, and tactical model choices. Use when domain terms, business rules, state transitions, policies, aggregates, workflow events, code/wiki terminology, or architecture boundaries need to be discovered, clarified, documented, or aligned before planning, implementation, tests, or refactoring.
---

# Domain-Driven Design

## Overview

Use Domain-Driven Design to make the software model match the domain the user is actually trying to change. The core discipline is strategic-first: discover language, boundaries, workflows, and invariants before choosing tactical patterns or implementation architecture.

DDD is useful when ambiguity in the domain would cause wrong names, wrong boundaries, brittle tests, accidental coupling, or over-engineered architecture. Good execution produces domain vocabulary, context boundaries, behavior rules, acceptance examples, and only the tactical patterns needed to protect real invariants.

`SKILL.md` is the workflow entry point. Load supporting references only when the current step needs more detail.

## When to Use

- A term has overloaded meanings across workflows, screens, APIs, modules, teams, tests, or docs
- The user is describing business behavior, not just technical plumbing
- A feature has hidden rules, state transitions, policies, eligibility checks, approvals, pricing, scheduling, permissions, accounting, or lifecycle constraints
- Architecture work depends on knowing the real domain seams before moving code or creating modules/services
- Tests need stable domain examples, invariants, and rejection cases before implementation
- A wiki/documentation layer needs vocabulary that maps to code behavior without copying framework or database names
- Existing code needs review because names, responsibilities, aggregates, contexts, or workflow boundaries feel confused
- Domain events have been identified and must be distinguished from commands, integration events, local callbacks, or event-sourcing decisions

**When NOT to use:** Don't use DDD for simple CRUD, thin admin panels, one-off scripts, styling, dependency setup, mechanical refactors, framework wiring, or bugs with an obvious technical reproduction path unless domain names or rules are ambiguous enough to cause defects. Use direct implementation, `diagnose`, `source-driven-development`, or `test-driven-development` instead.

## The Domain Modeling Workflow

```
1. SLICE     → Define the domain goal, capability, scope, actors, and evidence
2. LANGUAGE  → Build ubiquitous language from nouns, verbs, states, policies, and measures
3. BOUNDARY  → Find subdomains, bounded contexts, ownership, and translation seams
4. BEHAVIOR  → Surface commands, events, policies, state transitions, invariants, and examples
5. TACTICAL  → Choose the smallest model constructs that protect concrete domain rules
6. ARTIFACT  → Update durable domain docs only when evidence supports the model
7. HANDOFF   → Route to the next skill based on what the model revealed
8. VERIFY    → Prove the model is evidence-backed, scoped, and not pattern theater
```

### Step 1: Slice the Domain

Start with the smallest domain slice that can answer the user's request. Do not model the whole business unless the request truly depends on it.

```text
DOMAIN SLICE
- User goal:
- Business capability:
- In scope:
- Out of scope:
- Known actors:
- Existing artifacts to read:
- Decision needed now:
```

For an existing repo, inspect `CONTEXT.md`, `docs/domain-map.md`, `docs/domain-model.md`, ADRs, feature docs, tests, route/API names, schema names, and user-facing copy before inventing vocabulary.

Exit early if the slice is just mechanical implementation with no meaningful domain ambiguity.

### Minimum Useful DDD Pass

When the request has only a small amount of domain ambiguity, do not expand into a full model. Capture only:

- the disputed term or rule,
- the corrected meaning,
- the invariant or rejection case,
- one acceptance example,
- the next implementation or test handoff.

Stop there unless boundaries, workflows, or tactical patterns are actually needed.

### Step 2: Build the Ubiquitous Language

Extract domain nouns, verbs, states, events, policies, and measures. Separate code identifiers from domain terms; class names, table names, routes, and framework folders are evidence, not truth.

```text
UBIQUITOUS LANGUAGE TERM
- Term:
- Meaning:
- Context:
- Not the same as:
- Source/evidence:
- Status: stable | provisional | disputed
```

Rules:

- Prefer user-facing copy, tests, support tickets, product language, policies, and domain expert statements over implementation names.
- Mark disputed terms as provisional instead of making the report look certain.
- Update `CONTEXT.md` only when a term is stable enough to guide future work.
- Record open terminology questions when the answer affects naming, tests, boundaries, or irreversible architecture.

### Step 3: Discover Boundaries

Identify subdomains and bounded-context candidates. Prefer boundaries where language, lifecycle, rules, data ownership, consistency needs, or change cadence differ.

Use this compact boundary check before recommending architecture:

| Boundary Signal | Evidence to Look For |
|---|---|
| Different language | Same word means different things, or different words describe the same concept |
| Different rules | Policies, validation, permissions, state transitions, or invariants diverge |
| Different lifecycle | Concepts are created, changed, expired, archived, or reconciled differently |
| Different ownership | A team, module, system, or external provider owns the source of truth |
| Different consistency | One area needs atomic invariants while another can consume projections |
| Different change cadence | One area changes often while another should remain stable |

Load [STRATEGIC-DESIGN.md](STRATEGIC-DESIGN.md) for subdomain classification, bounded-context candidates, context maps, shared kernels, published language, and anti-corruption boundaries.

### Step 4: Surface Behavior and Invariants

Capture the behavior that the model must protect.

```text
DOMAIN BEHAVIOR
- Commands:
- Events/facts that happened:
- Actors:
- Policies/decisions:
- State transitions:
- Invariants:
- Rejection cases:
- Acceptance examples:
- Test seams:
```

Use past-tense events to model facts, not implementation messaging. Domain events do not automatically imply brokers, event sourcing, CQRS, or async integration.

Load [EVENT-STORMING.md](EVENT-STORMING.md) when workflow order, responsibilities, event timing, hidden business rules, or state transitions are unclear.

### Step 5: Choose Tactical Design Only When Needed

Model first. Pattern second. Code last. Choose the lightest tactical model that protects a concrete domain rule.

| Need | Prefer | Avoid Until Proven Necessary |
|---|---|---|
| Valid concept defined by attributes | Value object | Entity just because it has a name |
| Identity and lifecycle matter | Entity | Value-less data bag with procedural rules elsewhere |
| Multiple objects need one atomic consistency boundary | Aggregate | God aggregate or ORM graph aggregate |
| Cross-object decision that belongs to the domain | Policy/domain service | Application service with hidden business rules |
| Persist or retrieve an aggregate | Repository | Generic table gateway for every record |
| Fact that happened matters elsewhere | Domain event | Runtime integration event by default |
| Complex valid construction | Factory | Constructor ceremony for trivial objects |

Load [TACTICAL-DESIGN.md](TACTICAL-DESIGN.md) for value objects, entities, aggregates, domain services, policies, repositories, factories, and domain events.

### Step 6: Write or Update Artifacts

Create or update domain documentation only when there is enough evidence. Prefer small durable updates over large speculative docs.

Allowed artifacts:

- `CONTEXT.md` for stable vocabulary future agents should reuse
- `docs/domain-map.md` for subdomains, bounded contexts, and context relationships
- `docs/domain-model.md` for commands, events, policies, invariants, aggregates, and examples
- `docs/adr/*.md` for durable modeling decisions likely to be revisited

Load [ARTIFACTS.md](ARTIFACTS.md) for templates and artifact-specific rules.

Do not create ADRs for temporary priority, obvious naming, or implementation details that can be discovered from code.

### Step 7: Hand Off to the Next Skill

Route by what the model revealed, not by ceremony:

| Finding | Next Skill |
|---|---|
| Domain events may become runtime, integration, broker, stream, or replay contracts | `event-driven-architecture` |
| Bounded contexts imply module seams, refactoring, or architecture cleanup | `improve-codebase-architecture` |
| Invariants, examples, policies, or workflows need early proof | `shift-left-testing` |
| One behavior is ready for red-green-refactor | `test-driven-development` |
| Framework/library rules determine implementation choices | `source-driven-development` |
| Boundary, invariant, or modeling claim is high-stakes or uncertain | `doubt-driven-development` |
| A bug exposed domain misunderstanding | `diagnose` then `domain-driven-design` for the corrected model |

Use the shortest sequence that materially improves the result.

### Step 8: Verify the Model

For architecture-only work, produce a domain report. For implementation work, attach the report to the planning/testing handoff and make the invariants testable.

```text
DOMAIN-DRIVEN DESIGN REPORT
- Domain slice:
- Language: stable terms, overloaded terms, disputed terms, open questions
- Boundaries: core/supporting/generic subdomains, bounded contexts, context map notes
- Behavior: commands, events, policies, state transitions, invariants, rejection cases, acceptance examples
- Tactical design, if needed: value objects, entities, aggregates, services/policies, repositories, persistence/integration notes
- Artifact updates: created, updated, deferred, and why
- Rejected patterns/alternatives:
- Confidence and evidence:
- Handoff: next skill, first action, stop condition
```

If evidence is insufficient, state what is provisional and what question or source would settle it.

## DDD Defaults

Use these defaults unless the domain proves it needs more:

```text
Default to language before patterns.
Default to bounded contexts before services.
Default to direct CRUD when the domain is simple.
Default to value objects before entities when identity does not matter.
Default to small aggregates around consistency boundaries.
Default to domain events as modeling facts before runtime messages.
Default to artifact updates only when evidence is durable.
Default to open questions instead of fake certainty.
```

## See Also

Load supporting files only when needed:

- [STRATEGIC-DESIGN.md](STRATEGIC-DESIGN.md) — subdomains, bounded contexts, context maps, ownership, anti-corruption boundaries
- [EVENT-STORMING.md](EVENT-STORMING.md) — commands, events, actors, policies, read models, workflow order, open questions
- [TACTICAL-DESIGN.md](TACTICAL-DESIGN.md) — value objects, entities, aggregates, policies, repositories, factories, domain events
- [ARTIFACTS.md](ARTIFACTS.md) — `CONTEXT.md`, domain map, domain model, and ADR templates
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md) — pattern-first DDD, entity inflation, repository sprawl, false precision, CRUD denial

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We already have class/table names, so the domain language is known." | Code names are evidence, not truth. Check user-facing language, tests, policies, and domain examples before settling terminology. |
| "This is DDD, so we need aggregates, repositories, factories, and services." | Patterns are optional tools. Use them only when they protect concrete rules or consistency boundaries. |
| "Everything is an entity." | Identity over time must matter. Otherwise use a value object, read model, plain data, or no new type. |
| "The context map can match the folder structure." | Folders are weak evidence. Boundaries need semantic evidence: language, lifecycle, rules, ownership, consistency, or change cadence. |
| "Domain events mean we should use event sourcing or a broker." | Domain events are modeling facts. Runtime event architecture is a separate decision handled by `event-driven-architecture`. |
| "The model feels right, so we can present it as final." | Label confidence. Record provisional terms, alternatives, and open questions when evidence is incomplete. |
| "A shared domain package will keep everything consistent." | Shared kernels create coupling. Keep shared models tiny, stable, intentionally owned, and versioned; otherwise translate at boundaries. |
| "DDD is always better than CRUD." | CRUD is correct when the domain is simple and rules are thin. DDD effort belongs where ambiguity or business rules are load-bearing. |

## Red Flags

- The answer starts with aggregates, repositories, CQRS, event sourcing, or service boundaries before language and behavior are clear
- `SKILL.md` output copies class names, table names, route names, or folder names as business vocabulary without evidence
- Every noun becomes an entity or every table gets a repository
- One aggregate owns a whole workflow, unrelated entities, and cross-context rules
- Bounded contexts mirror teams, folders, or microservices without semantic evidence
- Domain events are immediately turned into broker topics, integration contracts, or event-sourced streams
- The report presents speculative boundaries, terms, or invariants as settled facts
- No rejection cases, invariants, state transitions, or acceptance examples are captured
- Artifact updates are large, confident, and unsupported by source evidence
- DDD is applied to mechanical CRUD or framework work where it adds ceremony but no correctness

## Verification

After using this skill:

- [ ] The domain slice is explicit, scoped, and tied to the user's actual decision or implementation need
- [ ] Ubiquitous language separates stable, provisional, and disputed terms with source/evidence
- [ ] Overloaded terms and unresolved terminology questions are documented when relevant
- [ ] Boundary recommendations cite language, lifecycle, rule, ownership, consistency, or change-cadence evidence
- [ ] Commands, events, actors, policies, state transitions, invariants, rejection cases, and acceptance examples are captured when behavior is in scope
- [ ] Tactical patterns are justified by concrete rules and simpler alternatives were rejected where relevant
- [ ] Domain events are not treated as runtime/integration events unless handed to `event-driven-architecture`
- [ ] Artifact updates are small, durable, and evidence-backed, or explicitly deferred
- [ ] The output identifies confidence level, open questions, and what would settle unresolved claims
- [ ] The handoff names the next skill only when it materially changes the result
- [ ] No unverifiable or pattern-first modeling claims remain
