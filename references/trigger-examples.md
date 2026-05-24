# Skill Trigger Examples

Use this file as a lightweight regression suite for whether the installed skills are reachable from normal user language. These are examples, not exhaustive rules. The router should still choose the smallest useful installed skill and avoid ceremony.

## Should trigger the router

| Prompt | Expected route |
|---|---|
| “Review this skill pack.” | `using-agent-skills` → likely `write-a-skill`, `doubt-driven-development`, or another specific review skill depending on the artifact |
| “Can this be made into a meaningful skill?” | `using-agent-skills` → `write-a-skill` plus `doubt-driven-development` when usefulness is uncertain |
| “Is this actually beneficial?” | `using-agent-skills` → `doubt-driven-development` when the decision has meaningful cost or uncertainty |
| “Be skeptical.” | `using-agent-skills` → `doubt-driven-development` for nontrivial artifacts or decisions |
| “Review your final judgment.” | `using-agent-skills` → `doubt-driven-development` |
| “Apply it.” | `using-agent-skills` → the skill implied by the previous accepted plan |
| “Plan it out first.” | `using-agent-skills` → the smallest planning skill matching the task, often `incremental-implementation`, `idea-refine`, or `write-a-skill` |
| “Is this production ready?” | `using-agent-skills` → likely `continuous-delivery-readiness`, `reliability-design`, `improve-codebase-architecture`, or `doubt-driven-development` depending on evidence needed |
| “Fix this bug and make sure it does not regress.” | `using-agent-skills` → `diagnose` and likely `test-driven-development` |
| “Make sure this config is valid according to the official docs.” | `using-agent-skills` → `source-driven-development` |
| “Check whether this causes a butterfly effect of bad decisions.” | `using-agent-skills` → `doubt-driven-development`; possibly `improve-codebase-architecture` for structural impact |
| “Refactor this code so it is easier to test.” | `using-agent-skills` → `improve-codebase-architecture` or `incremental-implementation` depending on scope |
| “This touches retries and duplicate requests.” | `using-agent-skills` → `reliability-design` |
| “The business logic or requirements are ambiguous.” | `using-agent-skills` → `domain-driven-design` |
| “The domain model, names, rules, or boundaries feel confused.” | `using-agent-skills` → `domain-driven-design` |
| “This feature has pricing, permissions, eligibility, lifecycle, approval, or policy rules.” | `using-agent-skills` → `domain-driven-design` |
| “This uses queues/events/outbox.” | `using-agent-skills` → `event-driven-architecture` |
| “This needs a webhook, background job, event notification, message handler, or async side effect.” | `using-agent-skills` → `event-driven-architecture` |
| “This workflow has eventual consistency, fan-out, retries, ordering, replay, or multiple consumers.” | `using-agent-skills` → `event-driven-architecture` |
| “Create a demo/prototype before committing to the design.” | `using-agent-skills` → `prototype` |

## Should not trigger the router by itself

| Prompt | Reason |
|---|---|
| “What is DDD?” | Simple explanation unless the user is applying it to an artifact/codebase |
| “What is event-driven architecture?” | Simple explanation unless the user is applying it to an artifact/codebase or evaluating an architecture decision |
| “Rewrite this sentence.” | Simple prose edit, no workflow needed |
| “Summarize this paragraph.” | Summarization, no skill routing needed unless the summary is a handoff artifact |
| “What does this term mean?” | Simple factual explanation unless it affects implementation or domain modeling |
| “Format this list alphabetically.” | Mechanical edit unless part of a broader artifact update |
| “Run the tests.” | Pure tooling step unless diagnosis, TDD, or release readiness is required |

## Anti-overtrigger checks

Before selecting a skill, ask:

1. Would this skill materially improve the result, or is it ceremony?
2. Is there a smaller skill that fits better?
3. Is the user asking for a simple answer, or for an artifact/decision/change?
4. Does the task need evidence, implementation slices, source verification, tests, reliability semantics, domain modeling, event contracts, or adversarial review?

If the answer is no, proceed normally.
