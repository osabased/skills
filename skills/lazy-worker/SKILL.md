---
name: lazy-worker
description: Guides the agent to reuse high-quality existing code, dependencies, tools, libraries, and project patterns before custom implementation, while rejecting low-trust or poor-fit reuse candidates. Use when a task may involve building new functionality, adding dependencies, choosing external projects, designing from scratch, or duplicating existing behavior.
---

# Lazy Worker

## Overview

Lazy Worker is a reuse-first engineering skill.

It prevents the agent from building things from scratch when trustworthy existing work can be reused, adapted, wrapped, or composed.

Lazy does **not** mean careless. It means:

> Be lazy about labor. Be strict about quality.

The agent should avoid unnecessary invention, but it must not adopt random low-quality projects just to avoid writing code. Bad reuse is worse than a small custom implementation.

Core rule:

> Reuse first. Quality-gate always. Build custom only when reuse is worse.

## When to Use

Use this skill when the task involves any of the following:

- building new functionality
- adding a new module, helper, abstraction, or subsystem
- choosing a library, package, GitHub repo, tool, MCP server, plugin, or framework
- implementing infrastructure-like behavior
- designing architecture
- duplicating behavior that may already exist
- solving a problem likely covered by existing tools
- researching projects to avoid building from scratch

Common examples:

- parsers
- file watchers
- search/indexing
- vector search
- validation
- schemas
- caching
- queues
- workflow engines
- state machines
- CLI tools
- adapters
- telemetry
- storage
- test utilities
- code generation
- project scaffolding

**When NOT to use:** Do not use this skill for tiny direct edits, pure explanations, formatting-only changes, or cases where the user explicitly wants a custom implementation with no reuse search. Do not perform external project research when local code, installed dependencies, or known project constraints already determine the right path.

Do not use this skill as an excuse to under-solve the task. The goal is less unnecessary work, not less correctness.

## Process

### 1. Name the Real Primitive

Before searching or building, identify the actual capability needed.

Bad:

> Build a file sync system.

Better:

> Need file watching, change detection, conflict handling, and a durable change ledger.

Bad:

> Build agent memory.

Better:

> Need indexed facts, provenance, freshness policy, and retrieval boundaries.

Do not search for vague product names when the task is really about smaller primitives.

### 2. Check Local Reuse First

Before creating anything new, inspect the current project for:

- existing modules
- existing helpers
- similar features
- project conventions
- tests
- scripts
- configs
- adapters
- prior implementations
- existing abstractions
- existing failure-handling patterns

Prefer the project’s existing boring pattern over a theoretically cleaner new dependency unless the current pattern is broken, unsafe, or clearly inadequate.

Local reuse usually beats external reuse because it is already integrated, already tested against the project’s assumptions, and easier to maintain.

### 3. Check Installed Reuse

Before adding a new dependency, inspect what is already available.

Check:

- package manifests
- lockfiles
- existing imports
- runtime/framework features
- standard library capabilities
- project scripts
- test tooling
- build tooling
- database extensions
- CLIs already used by the project

Do not add a new dependency for behavior that the project already has.

### 4. Check External Reuse Only When Needed

If local and installed options are insufficient, look for mature external options:

- reputable libraries
- official framework features
- standard protocols
- well-maintained CLI tools
- proven GitHub projects
- ecosystem-standard packages
- existing plugins or adapters

External reuse is useful only when it reduces total project complexity.

A dependency is not free. It adds update risk, security surface, license risk, transitive dependencies, API lock-in, and future maintenance burden.

### 5. Apply the Quality Gate

A reuse candidate is acceptable only if it is:

- fit for the actual primitive
- actively maintained or stable enough to trust
- meaningfully adopted, officially supported, or otherwise reputable
- inspectable
- compatible with the project’s license, runtime, and ecosystem
- smaller than the complexity it removes
- easy to isolate, replace, or remove
- not forcing unnecessary architecture changes

Reject candidates with:

- no visible adoption for nontrivial code
- unknown or untrusted maintainer
- deprecated, archived, or abandoned status
- stale commits/releases with unresolved issues
- no tests for meaningful behavior
- unclear or incompatible license
- suspicious install/build scripts
- huge dependency tree for a small task
- opaque or messy implementation
- architecture takeover
- poor fit disguised by a relevant README

A project having 0 stars is not always an automatic reject. A tiny, official, internal, or trivially inspectable package can still be acceptable. But a nontrivial external project with 0 stars, unknown maintainers, no users, no tests, and no trust trail should be rejected.

### 6. Choose the Lowest-Risk Path

Prefer solutions in this order:

1. Existing local code
2. Existing local pattern or convention
3. Existing installed dependency
4. Built-in framework/runtime feature
5. Mature external package, tool, protocol, or project
6. Thin adapter around reused capability
7. Small custom implementation with justification

Thin adapters are usually good. They let the project benefit from existing work without spreading third-party assumptions everywhere.

Custom implementation is acceptable when:

- reuse candidates are low quality
- existing options are too heavy
- dependency risk is worse than implementation cost
- integration cost exceeds build cost
- the needed behavior is small
- the behavior is core product logic
- the project needs tight control over semantics

When building custom, keep the implementation small and aligned with existing project patterns.

## Decision Rules

Do not build before checking reuse.

Do not add a dependency before checking installed dependencies.

Do not use a low-quality project just to avoid writing code.

Do not remodel the project around a dependency.

Do not duplicate existing local behavior.

Do not trust stars alone.

Do not reject local patterns just because a new package looks cleaner.

Do not adopt a project that solves a different problem than the actual primitive.

Do not create a subsystem when a wrapper, adapter, configuration change, or existing tool would solve the task.

## Rationalizations

### “It will be faster to just build it.”

Maybe. But first check whether the project already has the primitive, pattern, dependency, or framework feature. Building is only faster if reuse discovery and integration would cost more than the implementation.

### “I found a GitHub repo that does this.”

Finding a repo is not enough. It must pass the quality gate. Unknown, unmaintained, untested, low-adoption projects are not good reuse.

### “This package is popular, so it must be good.”

Popularity is a signal, not proof. Check fit, maintenance, dependency weight, license, API stability, and whether it forces architecture changes.

### “The local pattern is ugly; I should replace it.”

Not by default. Existing project patterns carry integration value. Improve or extend them unless they are clearly broken, unsafe, or blocking the task.

### “This deserves a new abstraction.”

Only if the abstraction reduces real complexity. Do not introduce a registry, plugin system, workflow engine, router, or framework when a small adapter or direct call is enough.

### “This dependency saves code.”

Saving lines is not the same as reducing complexity. A dependency is worth it only when it reduces total maintenance, risk, and implementation burden.

### “I should build the perfect version.”

Build the smallest correct version that satisfies the task and preserves future optionality. Do not overbuild in the name of future reuse.

## Red Flags

### Overbuilding Red Flags

- implementing a parser without checking parser libraries or existing parser usage
- implementing a file watcher without checking existing tools
- implementing search/indexing without checking existing database/search capabilities
- creating a new validation system when one exists
- creating a new workflow engine when a simpler task runner or state model works
- adding a new plugin system when the host already provides extension points
- writing duplicated helpers
- designing a large abstraction before checking project conventions
- replacing boring existing infrastructure with custom machinery

### Bad Reuse Red Flags

- 0 stars and no adoption trail for nontrivial code
- unknown maintainer with no trust signal
- no meaningful README or examples
- no tests
- no recent maintenance
- archived or deprecated repository
- unresolved critical issues
- unclear license
- suspicious install scripts
- huge dependency tree
- dependency forces a new architecture
- dependency solves only a small part while adding major complexity
- dependency is hard to remove once integrated
- package looks like a demo, toy, or abandoned experiment

## Verification

Before finalizing, verify:

- Local code and patterns were checked before new implementation.
- Installed dependencies and framework/runtime features were checked before adding anything new.
- External reuse candidates were quality-gated.
- The chosen option fits the actual primitive.
- The chosen option reduces total complexity rather than moving complexity into the dependency graph.
- Any new dependency is maintained, reputable, license-compatible, and reasonably isolated.
- Any custom implementation has a concrete justification.
- No unnecessary subsystem, abstraction, or architecture change was introduced.
- The solution follows existing project conventions.
- The result can be tested, replaced, or removed without major damage.

For implementation tasks, the final response should briefly state the reuse decision:

- what was reused
- what was rejected, if relevant
- why the chosen path is lower risk than the alternatives
- why custom code was necessary, if custom code was written

The agent should be able to answer:

> What already existed, why was it good enough or not good enough, and why is this the lowest-risk path?
