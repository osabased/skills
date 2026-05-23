---
name: continuous-delivery-readiness
description: Checks whether a code change remains safely releasable through the project's real delivery path. Use when work touches build, CI, packaging, deployment, release process, environment variables, feature flags, migrations, API/event compatibility, rollback safety, or production-facing behavior where "works locally" is not enough.
---

# Continuous Delivery Readiness

## Overview

Use Continuous Delivery readiness to keep the repository in a releasable state after each meaningful change. The core discipline is simple: do not stop at "the code is implemented"; prove the change can pass through the project's actual build, test, package, configuration, migration, and release path without hidden manual steps or avoidable production risk.

This is not a generic DevOps checklist. Do not invent CI, deployment scripts, cloud infrastructure, feature flags, or release documentation just because they sound mature. Start from the repository's real delivery surface and improve only the part that materially protects the current change.

Good execution produces a small release-readiness note: what delivery path exists, what the change affects, what gates were run or added, what remains manual, and whether the change is safe to merge/release now.

## When to Use

- A task changes build, test, lint, typecheck, package, Docker/container, CI, deployment, or release scripts
- A feature/refactor/migration must be safe to merge even if it is not immediately exposed to users
- The change touches environment variables, secrets, config loading, runtime flags, feature flags, database migrations, background jobs, public APIs, event contracts, or external integrations
- A plan claims a change is "production-ready", "safe to deploy", "releasable", "behind a flag", "backward compatible", or "rollback-safe"
- A multi-slice implementation needs to decide what must be verified before merge versus before release
- A bug fix revealed missing release gates, manual deployment assumptions, or environment drift

**When NOT to use:** Do not use this for tiny local-only edits, pure prose/docs, isolated tests, throwaway prototypes, or code that cannot affect build/runtime/release behavior. Use `incremental-implementation` for small verified coding slices, `shift-left-testing` for choosing early feedback gates, and `source-driven-development` for framework/library correctness. Use this skill only when releasability or delivery-path safety changes the work.

## The Releasable Change Workflow

```text
1. DISCOVER  → Find the project's actual delivery path
2. CLASSIFY  → Identify what the change can break after local implementation
3. GUARD     → Choose the smallest release protection that materially reduces risk
4. VERIFY    → Run or specify evidence for merge/release readiness
5. RECORD    → Leave a compact release-readiness note
```

### Step 1: Discover the Actual Delivery Path

Inspect the repository before recommending delivery changes. Look for real, existing signals:

- package/build scripts: `package.json`, `Makefile`, task runners, language-specific project files
- quality gates: test, lint, typecheck, formatting, integration/e2e commands
- CI definitions: `.github/workflows`, GitLab CI, CircleCI, Buildkite, local scripts, pre-commit hooks
- packaging/runtime: Dockerfiles, compose files, Procfiles, serverless manifests, binaries, artifacts
- deployment/release docs: `README`, `docs/`, `release.md`, changelog, versioning, migration notes
- config/environment surface: `.env.example`, config schemas, secret names, feature flags, runtime defaults
- data/integration boundaries: migrations, queues, jobs, event schemas, public APIs, external service contracts

If no delivery path exists, do not fabricate one. State the missing path and choose the smallest local proof available for the current change.

### Step 2: Classify Release Impact

Name the impact category before acting:

| Impact | Questions to answer |
|---|---|
| Build/package | Does the app still compile, bundle, package, or produce expected artifacts? |
| Test gate | Which existing check would catch a bad version of this change? |
| Runtime config | Are new env vars, defaults, secrets, flags, or config schemas documented and safe? |
| Data migration | Is the migration ordered, repeatable, backward-compatible, and reversible where practical? |
| API/event contract | Can old and new producers/consumers coexist during rollout? |
| User exposure | Is incomplete behavior hidden behind a safe flag/default? |
| Rollback | Can the change be reverted without data loss, stuck jobs, or incompatible clients? |
| Observability | Would failures be visible through logs, metrics, traces, health checks, or explicit errors? |

Prefer the smallest set of categories that actually apply. Do not run a full release audit for a small code change unless the blast radius justifies it.

### Step 3: Guard the Release Boundary

Choose the lightest guard that makes the change releasable:

- Existing command is enough: run it and record the result.
- Existing command exists but is too broad/slow: run the closest relevant subset and name the unrun gate.
- No gate covers the risk: add or propose the smallest deterministic gate that would catch the failure.
- Behavior is incomplete: keep it unexposed by default through a safe flag, config default, route guard, or unreachable path.
- Data shape changes: prefer expand/contract migrations, additive fields, compatibility windows, and explicit backfill/rollback notes.
- Public contracts change: preserve backward compatibility or version the contract; do not break consumers silently.
- Deployment requires manual work: document the exact manual step and the failure if it is skipped.

Do not add new CI/CD infrastructure unless the current change has a concrete, repeated failure mode that local verification cannot reliably catch.

### Step 4: Verify Merge/Release Readiness

Separate local evidence from merge/release evidence:

```text
LOCAL EVIDENCE
- Commands run:
- Result:
- Manual check, if any:

MERGE/RELEASE EVIDENCE
- Existing CI/release gate:
- New or changed gate:
- Required manual step:
- Known gap or risk:
```

If a command cannot be run in the current environment, state that plainly and provide the exact command the next agent/human should run. Do not claim release readiness from static inspection alone.

### Step 5: Record the Release-Readiness Note

For any non-trivial use of this skill, end with a compact note:

```text
CONTINUOUS DELIVERY READINESS
Delivery path found: [scripts/CI/deploy docs or "none found"]
Change impact: [categories]
Guards used/added: [commands/tests/flags/migrations/contracts]
Safe to merge: yes/no/conditional because [reason]
Safe to release: yes/no/conditional because [reason]
Follow-up: [only if materially needed]
```

Keep the note factual. Do not call something production-ready if release evidence is missing.

## Relationship to Other Skills

- Use `shift-left-testing` before this when the right quality gate or test seam is unclear.
- Use `incremental-implementation` to land the code in small verified slices; use this skill to check whether those slices remain releasable.
- Use `test-driven-development` for behavior proof inside a slice.
- Use `event-driven-architecture` for event contract, idempotency, ordering, replay, and consumer compatibility decisions; then use this skill to verify rollout/release readiness.
- Use `reliability-design` for non-event retries, timeouts, idempotency, duplicate handling, partial failure, concurrency, caching, background job recovery, or reconciliation semantics before judging releasability.
- Use `source-driven-development` when CI, deployment, framework, or platform-specific behavior depends on current official documentation.
- Use `doubt-driven-development` when the release-safety claim is high-stakes or not directly proven by commands/tests.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Tests passed locally, so this is releasable." | Local tests are necessary, not sufficient, when packaging, config, migrations, CI, deploy steps, or runtime contracts are affected. |
| "I'll add CI/deploy automation while I'm here." | New delivery infrastructure is risky scope expansion unless it directly protects the current change and can be verified. |
| "The flag can come later." | If incomplete behavior can be reached by users or consumers, the safe default belongs in the same slice. |
| "Rollback is just git revert." | Reverts do not automatically reverse migrations, external side effects, queued jobs, published events, or client-visible contracts. |
| "The deploy process is obvious." | If a release depends on undocumented manual steps, it is not obvious to the next agent or operator. |

## Red Flags

- Adding generic CI/CD files without inspecting existing scripts, repo conventions, or deployment path
- Claiming "production-ready" without build/test/package/config evidence
- New env vars or secrets without examples, defaults, validation, or deployment notes
- Migration changes with no ordering, compatibility, backfill, or rollback story
- Public API or event contract changes without consumer compatibility analysis
- Feature flags that default to exposed/on for incomplete behavior
- Manual deployment steps hidden in prose rather than named as required release work
- Running the same broad check repeatedly without connecting it to the specific release risk
- Treating Continuous Delivery as a reason to build a platform instead of protecting the current change

## Verification

After using this skill:

- [ ] The actual delivery path was inspected or explicitly reported as missing
- [ ] The change's release-impact categories were named
- [ ] Local verification commands and results were recorded, or exact unrun commands were provided
- [ ] Merge/release gates, manual steps, and known gaps were separated
- [ ] New config, migrations, contracts, flags, or deployment assumptions were guarded or documented when relevant
- [ ] Safe-to-merge and safe-to-release claims are evidence-backed and appropriately conditional
- [ ] No generic CI/CD, deploy, or infrastructure work was added without a concrete benefit
