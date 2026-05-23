---
name: improve-codebase-architecture
description: Find deepening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, or make a codebase more testable and AI-navigable.
---

# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.

## Glossary

Use these terms exactly in every suggestion. Consistent language is the point — don't drift into "component," "service," "API," or "boundary." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Use this, not "boundary.")
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

Key principles (see [LANGUAGE.md](LANGUAGE.md) for the full list):

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.**
- **One adapter = hypothetical seam. Two adapters = real seam.**

This skill is _informed_ by the project's domain model. The domain language gives names to good seams; ADRs record decisions the skill should not re-litigate. If domain language, bounded contexts, or invariants are unclear, run `domain-driven-design` before proposing architecture changes.

## Process

### 1. Explore

Read the project's domain glossary and any ADRs in the area you're touching first.

Then launch an exploration subagent using the IDE's subagent mechanism. If named subagent types are supported, use the closest equivalent to an `Explore` reviewer; otherwise use a generic codebase-exploration subagent. Don't follow rigid heuristics — explore organically and note where understanding the code creates friction.

Use this brief when the IDE needs an explicit subagent prompt:

```text
Explore the target area as a codebase cartographer. Find architecture-improvement candidates that would make the code easier to understand, change, test, or document.

Focus on:
- shallow modules whose interfaces are nearly as complex as their implementations
- concepts split across too many files
- hidden coupling, duplicated domain language, or unclear seams
- places where a deeper module or clearer boundary would reduce user/agent cognitive load
- evidence: specific files, symbols, call paths, tests, and config

Do not propose broad cleanup. Return 3-5 concrete candidates with evidence and expected user-visible benefit.
```

While reviewing the findings, ask:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow** — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts of the codebase are untested, or hard to test through their current interface?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp directory through the runtime environment instead of hardcoding one: use `$TMPDIR`/`$TEMP`/`$TMP` on POSIX shells when available, `%TEMP%` on Windows, or the host language's standard temp-directory API. Write to `architecture-review-<timestamp>.html` so each run gets a fresh file. If the environment supports opening local files, open it; otherwise tell the user the absolute path.

The report must work without network access. Use embedded CSS and inline SVG/HTML diagrams by default. Mermaid is optional only when the environment already supports it locally or the user accepts CDN use. Each candidate gets a **before/after visualisation**. Be visual.

For each candidate, render a card with this template:

- **Files** — which files/modules are involved
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and how tests would improve
- **Before / After diagram** — side-by-side, custom-drawn, illustrating the shallowness and the deepening
- **Recommendation strength** — one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with a **Top recommendation** section: which candidate you'd tackle first and why.

**Use CONTEXT.md vocabulary for the domain, and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.** If `CONTEXT.md` defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service."

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly in the card (e.g. a warning callout: _"contradicts ADR-0007 — but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.

Do NOT propose interfaces yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Design challenge loop

Once the user picks a candidate, run a direct design challenge conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept not in `CONTEXT.md`?** Add the term to `CONTEXT.md` using the local `CONTEXT.md` guidance below. Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `CONTEXT.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing — skip ephemeral reasons ("not worth it right now") and self-evident ones. Use the local ADR guidance below.
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).

### Local documentation guidance

Use this guidance when the architecture review needs to record durable project knowledge. Prefer the project's existing documentation conventions when they already exist; otherwise use the local formats below.

#### `CONTEXT.md` guidance

`CONTEXT.md` is lightweight project memory for stable domain language and architectural vocabulary. It is not a scratchpad, meeting transcript, or place to record speculative ideas as accepted truth.

Add or update `CONTEXT.md` only for concepts that will help future explorers name seams, modules, invariants, or flows correctly. Keep entries short and evidence-backed.

Recommended shape:

```md
## <Concept Name>

Meaning: <one or two sentences defining the concept in project language.>

Invariants:
- <durable rule, constraint, or lifecycle fact callers/modules must preserve.>

Related modules/files:
- `<path>` — <how this file/module relates to the concept.>

Open questions:
- <only if the question affects architecture choices. Omit if none.>
```

Rules for `CONTEXT.md` updates:

- Use the user's/project's domain terms when they exist; do not invent polished names over established language.
- Preserve literal code names when referring to existing files, modules, symbols, or tests.
- Mark uncertainty as an open question instead of writing guesses as vocabulary.
- Do not add a term merely because a refactor idea sounds attractive. Add it because the term names real project behavior, a real invariant, or a real seam.

#### ADR guidance

An ADR records a durable architecture decision so future reviews do not re-litigate the same issue. Create one only when the decision is load-bearing: it changes module shape, rejects an attractive refactor, protects an integration constraint, or explains a tradeoff future agents are likely to question.

Use the repo's existing ADR location and numbering if present. If no convention exists, use `docs/adr/` and a clear filename such as `0001-keep-order-validation-in-ingress.md` or `YYYY-MM-DD-keep-order-validation-in-ingress.md`.

Recommended shape:

```md
# ADR: <Decision Title>

Date: <YYYY-MM-DD>
Status: Proposed | Accepted | Superseded

## Context

<What architectural pressure, friction, constraint, or rejected candidate led to this decision?>

## Decision

<The decision in one or two direct paragraphs. For rejected candidates, state what will not be changed and why.>

## Consequences

- <Positive consequence.>
- <Negative or accepted tradeoff.>
- <Operational/testing/documentation implication.>

## Alternatives considered

- <Alternative> — <why it was rejected or deferred.>

## Review trigger

<What future evidence would justify revisiting this ADR?>
```

Rules for ADRs:

- Do not create ADRs for temporary priorities, vague preferences, or obvious facts.
- For a rejected architecture candidate, capture the specific reason that prevents re-suggestion later.
- Tie the decision to observed code, tests, runtime constraints, domain invariants, or user-stated priorities.
- Include a review trigger when the decision depends on scale, usage, ownership, or an external dependency changing.

