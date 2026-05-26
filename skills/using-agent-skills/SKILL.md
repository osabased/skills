---
name: using-agent-skills
description: Routes nontrivial coding, debugging, planning, design, review, testing, documentation, architecture, reliability, release, research, or skill-authoring, reuse, or dependency-selection work to the smallest useful installed skill. Use at the start of any nontrivial user request involving a codebase, agent workflow, implementation plan, bug, refactor, architecture decision, test strategy, source-sensitive library/framework behavior, release readiness, prototype, dependency choice, external project selection, reuse decision, or skill-pack change. Also use when the user asks whether something is useful, production-ready, reliable, overengineered, under-specified, risky, or worth changing.
---

# Using Agent Skills

## Purpose

Router for the local `skills/` pack. Pick the smallest installed workflow that materially improves the result, apply it deliberately, and stop when extra skill use would become ceremony.

This pack supports idea refinement, domain modeling, event-driven design, prototyping, source-grounded implementation, reliability semantics, reuse-first implementation, early quality gates, TDD, incremental implementation, CD/release readiness, diagnosis, architecture improvement, adversarial review, skill authoring, and session handoff.

Do not route to absent skills. If no installed skill applies, proceed normally and name any useful-but-absent skill only when that matters.

## Routing Rules

Use the table as the primary router. Chain skills only when the next skill changes a decision, artifact, test, implementation slice, release judgment, or stop condition.

### Trigger on normal work language

Do not wait for the user to say “use a skill,” “choose a workflow,” or name a skill. This router exists to catch ordinary work requests and select the smallest useful installed skill.

Common prompts that should trigger this router when the task is nontrivial:

- “Review this,” “audit this,” “is this good,” “is this production ready,” or “is this actually beneficial?”
- “Be skeptical,” “be honest,” “review your final judgment,” or “make sure this does not cause bad decisions.”
- “Plan it out,” “apply it,” “fix this bug,” “implement this,” “refactor this,” or “make this reliable.”
- “Check the official docs,” “make sure this config is valid,” or “verify the current syntax/API.”
- “Don’t build from scratch,” “find an existing project,” “reuse this,” “add a dependency,” or “is this library/repo worth using?”
- “Create/revise/package this skill,” “review this skill pack,” or “can this be made into a meaningful skill?”

Do not over-trigger for casual explanations, simple rewrites, tiny formatting edits, or tasks where a workflow would add more ceremony than value.

| Situation / Intent | Use | Primary Output |
|---|---|---|
| Raw, vague, early, or under-tested idea; needs options, assumptions, tradeoffs, red flags, or sharper scope. | `idea-refine` | Refined concept with problem statement, MVP scope, assumptions, red flags, and open questions. |
| User wants an early plan challenged, pressure-tested, or stress-tested. | `idea-refine` Direct Challenge Mode | Direct challenge of assumptions, weak spots, alternatives, and decision pressure. |
| Domain language, business rules, bounded contexts, subdomains, invariants, workflows, or code/wiki terminology are load-bearing. | `domain-driven-design` | Domain model report and updates to `CONTEXT.md`, `docs/domain-map.md`, `docs/domain-model.md`, or ADRs when supported. |
| Runtime/integration events, async workflows, pub/sub, queues, streams, event buses, outbox/inbox, sagas, projections, replay, event contracts, fan-out, or event reliability need design. | `event-driven-architecture` | EDA decision report with event map, topology, contracts, reliability plan, observability, tests, and artifact updates when supported. |
| Need to try, mock up, sanity-check, or compare a design before committing. | `prototype` | Throwaway logic prototype or UI variants answering a specific design question. |
| Task may involve building from scratch, adding a dependency, choosing an external project/tool, duplicating existing behavior, or designing reusable infrastructure where local/installed reuse may exist. | `lazy-worker` | Reuse decision: local/installed/framework/external options checked, low-trust candidates rejected, and custom work justified only when reuse is worse. |
| Framework/library behavior is version-sensitive, unfamiliar, likely stale, production-critical, reusable, or explicitly needs source-backed correctness. | `source-driven-development` | Mandatory targeted authoritative source checks and citations for the decisions sources actually affect. |
| Multi-file work has behavior, dependencies, rollback, validation, or boundary risk that would be hard to reason about in one pass. | `incremental-implementation` | Story card when useful, ordered slices, per-slice implementation, verification, and commit-ready increments. |
| Non-event behavior involves retries, timeouts, idempotency, duplicate inputs, external services, background jobs, concurrency, caching/staleness, partial failure, recovery, or reconciliation. | `reliability-design` | Failure modes, semantics, observability, tests, and handoff to quality gates or TDD. |
| Feature, refactor, migration, or risky change needs early quality feedback, test seams, risk map, or gates before implementation. | `shift-left-testing` | Risk map, test seams, traceability-lite, minimum local/CI gate, explicit non-goals, and execution handoff. |
| Change touches build, CI, packaging, deployment, release process, env vars, feature flags, migrations, public contracts, or production-facing behavior. | `continuous-delivery-readiness` | Delivery-path discovery, release-impact classification, merge/release gates, rollback/config notes, and safe-to-merge/release judgment. |
| User asks for TDD, red-green-refactor, behavior-first tests, integration tests, or test-first bug fixes/features. | `test-driven-development` | One behavior at a time: failing test, minimal implementation, refactor, repeat. |
| Something is broken, failing, throwing, slow, flaky, or needs root-cause analysis. | `diagnose` | Reproduction loop, ranked hypotheses, instrumentation, fix, and regression test. |
| User wants architectural improvement, refactoring opportunities, better testability, lower coupling, better seams, or more AI-navigable structure. | `improve-codebase-architecture` | Architecture improvement findings informed by project language and ADRs. |
| Costly, uncertain, boundary-crossing, weakly-evidenced, or hard-to-test claim/decision needs adversarial review before it stands. | `doubt-driven-development` | Fresh-context challenge of artifact and contract, followed by reconciliation. |
| User wants to create, revise, or package a skill. | `write-a-skill` | Valid skill folder structure and `SKILL.md` with progressive disclosure. |
| User needs the current conversation compacted for another agent/session. | `handoff` | Handoff document with context, decisions, evidence, gaps, suggested installed skills, stop condition, and sensitive-data redaction. |

## Branching and Sequence Rules

Prefer the shortest sequence that fits. These are allowed escalations, not mandatory pipelines.

- **Idea path:** `idea-refine` → `prototype` only when empirical exploration would answer a real design question.
- **Domain path:** `domain-driven-design` → `event-driven-architecture` when domain events become runtime/integration events; → `improve-codebase-architecture` when boundaries imply module/seam changes; → `shift-left-testing` when invariants need proof.
- **EDA path:** `event-driven-architecture` → `domain-driven-design` first if language/boundaries are unclear; → `source-driven-development` when broker/framework specifics matter; → `shift-left-testing` for contract/reliability gates; → `test-driven-development` for the first behavior.
- **Prototype path:** `prototype` → capture the validated decision; then use `domain-driven-design`, `shift-left-testing`, `source-driven-development`, `incremental-implementation`, or `test-driven-development` only as the real implementation requires.
- **Reliability path:** `reliability-design` → `event-driven-architecture` when the issue is actually event contracts, consumers, replay, DLQs, or topology; → `shift-left-testing` when semantics need gates; → `test-driven-development` for the first behavior.
- **Quality path:** `shift-left-testing` → `reliability-design` if failure semantics are undefined; → `test-driven-development` once the first behavior is ready for red-green proof.
- **Implementation path:** `lazy-worker` first when the work may be solved by local code, installed dependencies, framework/runtime features, or mature external projects instead of custom implementation. Check local and installed reuse before external research; skip external project search when local code, installed dependencies, or known constraints already determine the right path. Add `source-driven-development` when the selected reuse path depends on version-sensitive, unfamiliar, reusable, or production-critical framework/library behavior; once selected, authoritative source use is mandatory → `incremental-implementation` when the work needs meaningful slices → `test-driven-development` inside slices when behavior proof is needed → `continuous-delivery-readiness` when release/deploy/config/migration safety matters.
- **Bug path:** `diagnose` → deterministic reproduction → fix → regression test; add `shift-left-testing` if the bug reveals a missing gate, `continuous-delivery-readiness` if delivery/config drift is involved, and `doubt-driven-development` if the fix rests on costly, uncertain, boundary-crossing, or weakly-evidenced assumptions.
- **Architecture path:** `domain-driven-design` when names/boundaries/rules are unclear → `improve-codebase-architecture` → `shift-left-testing` for seam/gate strategy → `source-driven-development` for framework/library changes that need source verification → `test-driven-development` for behavior-preserving proof.
- **Risk path:** state the claim → `doubt-driven-development` when wrongness would be costly or hard to catch directly → reconcile → proceed, revise, or stop.
- **Continuity path:** `handoff` preserves decisions, evidence, gaps, stop condition, and next suggested installed skills.

## Operating Rules

1. **Smallest useful skill wins.** Do not chain for ceremony. A simple failing test does not need ideation; a risky migration may need source verification, TDD, CD readiness, and adversarial review.
2. **Prefer trustworthy reuse before invention.** When work could involve new code, new abstractions, dependencies, tools, or external projects, use `lazy-worker` to check local/installed/framework reuse before building from scratch or adopting low-trust projects.
3. **State consequential assumptions.** For non-trivial work, surface assumptions affecting scope, architecture, safety, or user-visible behavior. Pause on assumptions that are likely wrong or expensive to reverse unless the user asked you to proceed autonomously.
4. **Build or find a feedback loop.** Implementation, diagnosis, and architecture changes need evidence. Prefer unit/integration tests, scripted harnesses, HTTP/curl checks, browser automation, snapshots/diffs, then manual notes as a last resort.
5. **Verify behavior, not vibes.** Completion requires task-appropriate evidence: passing tests, reproduced bug fixed, prototype question answered, official docs cited, delivery path checked, or adversarial findings reconciled.
6. **Keep scope surgical.** Do not refactor adjacent systems, delete unknown code, rewrite unrelated files, or broaden the task because an opportunity is visible. Use architecture improvement only when requested or necessary for safe completion.
7. **Push back on bad plans.** Name concrete risks and provide a better alternative. If the user knowingly overrides, continue within safe bounds.

## Skill Loading Rules

1. Read the selected skill's `SKILL.md` before applying it.
2. Load support files only when they materially improve the current task.
3. Inspect scripts before running them.
4. Do not invent missing companion skills, phases, or agents.
5. If two skills conflict, follow the more specific skill and explain the tradeoff.
6. If `source-driven-development` is selected but no authoritative source is fetched and used, treat that as a routing or execution failure, not a successful source-driven pass.

## Support Files

| Skill | Support Files |
|---|---|
| `diagnose` | `scripts/hitl-loop.template.sh` for human-in-the-loop reproduction loops. |
| `domain-driven-design` | `STRATEGIC-DESIGN.md`, `TACTICAL-DESIGN.md`, `EVENT-STORMING.md`, `ARTIFACTS.md`, `ANTI-PATTERNS.md`. |
| `event-driven-architecture` | `TOPOLOGY.md`, `EVENT-CONTRACTS.md`, `RELIABILITY.md`, `ARTIFACTS.md`, `ANTI-PATTERNS.md`. |
| `idea-refine` | `examples.md`, `frameworks.md`, `refinement-criteria.md`, `scripts/idea-refine.sh`. |
| `lazy-worker` | None. |
| `improve-codebase-architecture` | `DEEPENING.md`, `HTML-REPORT.md`, `INTERFACE-DESIGN.md`, `LANGUAGE.md`. |
| `prototype` | `LOGIC.md`, `UI.md`. |
| `test-driven-development` | `tests.md`, `mocking.md`, `interface-design.md`, `deep-modules.md`, `refactoring.md`. |
| `write-a-skill` | `TEMPLATE.md`, `EVALUATION.md`. |
| `incremental-implementation`, `reliability-design`, `continuous-delivery-readiness`, `shift-left-testing`, `handoff` | None. |

## Failure Modes to Avoid

1. Routing to absent skills from another pack.
2. Using `doubt-driven-development` for trivial edits, routine local choices, or directly proven changes.
3. Letting `prototype` become the permanent implementation path.
4. Writing version-sensitive, unfamiliar, reusable, or production-critical framework/library code from memory when official docs should be checked.
5. Selecting `lazy-worker` but still building from scratch without checking local code, installed dependencies, framework/runtime features, and reasonable external reuse options.
6. Selecting `lazy-worker` and adopting a low-trust dependency or random GitHub project just to avoid implementation.
7. Selecting `source-driven-development` but not fetching and using authoritative sources.
8. Writing a batch of tests before the first red-green loop proves the path.
9. Treating shift-left testing as permission to test private implementation details early.
10. Diagnosing without reproduction or a feedback loop.
11. Treating architecture improvement as permission for broad cleanup.
12. Using DDD as pattern theater instead of evidence-backed domain modeling.
13. Using EDA as decoupling theater instead of proving async/fan-out/reliability value.
14. Creating handoffs that duplicate existing artifacts instead of linking to them.
15. Using `reliability-design` as a generic checklist instead of concrete non-event failure semantics.
16. Writing bloated new `SKILL.md` files instead of using progressive disclosure.
17. Claiming completion without evidence.
18. Treating Continuous Delivery as CI/CD theater instead of inspecting the real delivery path.
19. Using `incremental-implementation` story cards or slice rituals for trivial mechanical edits.
