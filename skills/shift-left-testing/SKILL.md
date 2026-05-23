---
name: shift-left-testing
description: Designs early quality feedback before implementation. Use when starting a feature, refactor, migration, risky change, or test strategy and the goal is to catch likely failures earlier by choosing test seams, acceptance examples, local checks, CI gates, and explicit non-goals before writing code.
---

# Shift-Left Testing

Design the earliest useful evidence for a change before implementation begins. This is not TDD: it decides what should be proven, where proof belongs, and which feedback loop should exist first. Then `test-driven-development` can execute red-green-refactor.

## When to Use

Use when starting a non-trivial feature, refactor, migration, integration, risky behavior change, test strategy, quality gate, acceptance-criteria pass, or early risk-reduction pass.

Use after `diagnose` when a bug reveals a missing regression gate. Use after `reliability-design` when failure semantics need tests. Use with `improve-codebase-architecture` when no reliable test seam exists.

Do not use for tiny edits with an obvious existing test command, pure diagnosis of an active bug, or one-test-at-a-time implementation already covered by `test-driven-development`.

## Core Rule

Shift left does **not** mean "write every test early." It means move the cheapest reliable feedback to the earliest point where it can change the design.

Prefer minimal, high-signal gates over comprehensive theater. Avoid tests coupled to implementation details, speculative mocks, and huge test plans that slow implementation without reducing real risk.

## Process

### 1. Identify change and failure cost

```text
CHANGE: [user-visible or system-visible behavior]
FAILURE COST: [what breaks if this is wrong]
RISK LEVEL: low | medium | high
```

If behavior is vague, hand off to `idea-refine` before testing strategy.

### 2. Map likely failures

List the 3-7 most likely or expensive failure modes. Include product, data, integration, security, performance, accessibility, and migration risks only when relevant.

### 3. Choose test seams

For each risk, choose the earliest reliable seam: pure function/module test, public API/service interface test, integration test through real adapters, CLI/scripted harness, HTTP/curl check, browser/e2e smoke, static/type/lint/security check, or manual verification only when automation is not worth it.

The seam must exercise the real behavior path. If no good seam exists, say so and route to `improve-codebase-architecture` when worthwhile.

### 4. Define the minimum gate

```text
MINIMUM GATE
- Acceptance examples: [1-5 observable examples]
- First runnable check: [exact command or proposed harness]
- Required before implementation: [yes/no and why]
- Required before merge/release: [local, CI, manual, none]
```

For non-trivial or reliability-sensitive behavior, add traceability-lite so the gate proves the behavior that matters instead of merely adding tests:

```text
TRACEABILITY
- Requirement / behavior:
- Acceptance example:
- Failure mode:
- Test seam:
- Evidence command:
- Artifact touched:
```

Local-first is the default. Recommend CI only when deterministic, not too slow, and protective against mistakes local-only validation would miss.

### 5. Decide what not to test

Name exclusions so the plan does not bloat. Do not test private methods, internal call ordering, framework internals, or structure that should remain free to refactor.

### 6. Handoff to execution

End with the next skill: `source-driven-development` for framework/library behavior, `reliability-design` when retries/timeouts/idempotency/partial failure semantics are still undefined, `test-driven-development` for the first red-green behavior, `diagnose` when a current failure still lacks reproduction, `improve-codebase-architecture` when no reliable test seam exists, or normal execution when the gate is a simple command/checklist.

## Output Format

```text
SHIFT-LEFT TEST PLAN
CHANGE
- ...
RISK MAP
1. ...
TEST SEAMS
- Risk: ...
  Seam: ...
  Why this seam: ...
MINIMUM QUALITY GATE
- First check:
- Local command:
- CI recommendation:
- Manual check, if any:
TRACEABILITY, IF NON-TRIVIAL
- Requirement / behavior:
- Acceptance example:
- Failure mode:
- Test seam:
- Evidence command:
- Artifact touched:
DO NOT TEST YET
- ...
EXECUTION HANDOFF
- Next skill:
- First action:
- Stop condition:
```


## Verification

After using this skill:

- [ ] Change and failure cost are named
- [ ] Risk map focuses on likely or expensive failures, not every possible test
- [ ] Test seams exercise real behavior paths through stable/public interfaces
- [ ] Minimum gate includes first runnable check or exact proposed harness
- [ ] Traceability-lite is included for non-trivial or reliability-sensitive behavior
- [ ] Explicit non-goals prevent test-plan bloat
- [ ] Next skill or execution step is named
