# DDD Anti-Patterns

Check this file before making strong modeling recommendations.

## Pattern-first DDD

Symptom: The answer starts with entities, repositories, aggregates, factories, CQRS, or event sourcing before the domain language is clear.

Correction: Return to language, behavior, boundaries, and invariants. Patterns are allowed only when they protect a concrete rule.

## Entity inflation

Symptom: Every noun becomes an entity or class.

Correction: Ask whether identity over time matters. If not, prefer value object, plain data, read model, or no new type.

## Repository everywhere

Symptom: Every table or external API gets a repository because the pattern feels DDD-ish.

Correction: Use repositories for aggregate persistence/lookup when the domain model benefits. Use adapters/query models for reads and integration plumbing.

## Anemic domain theater

Symptom: Domain objects have names but no behavior or invariant protection; all rules live in procedural application services.

Correction: Move rules to the object/policy/aggregate that owns the invariant, or admit the model is just a transaction script if the domain is simple.

## God aggregate

Symptom: One aggregate owns a whole workflow, many unrelated entities, large object graphs, and every rule.

Correction: Split by consistency boundary. Reference other aggregates by identity. Use process managers/policies for cross-aggregate workflows.

## Event sourcing by default

Symptom: Every event storming output becomes an event-sourced architecture.

Correction: Domain events are modeling facts. Event sourcing is an implementation choice that requires audit/replay/versioning value high enough to justify operational cost.

## Context map as org chart

Symptom: Bounded contexts mirror teams, folders, or services without checking language/rule differences.

Correction: Boundaries need semantic evidence: different meanings, ownership, lifecycle, consistency, or change cadence.

## Shared kernel sprawl

Symptom: Many contexts share a growing common domain package.

Correction: Shared kernels should stay tiny, stable, versioned, and intentionally co-owned. Otherwise translate at the boundary.

## False precision

Symptom: The report presents speculative terms, aggregates, or boundaries as settled facts.

Correction: Label confidence. Record open questions. Ask domain questions when the answer affects irreversible architecture.

## CRUD denial

Symptom: A simple admin workflow gets over-modeled because DDD is available.

Correction: Use CRUD when CRUD is honest. Spend DDD effort where the domain has rules, ambiguity, or strategic value.

## Code-name leakage

Symptom: The domain glossary copies class names, table names, or framework route names as if they were business language.

Correction: Separate code identifiers from domain terms. Use user-facing copy, tests, policies, and domain expert language as stronger evidence.
