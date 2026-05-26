# Lazy Worker Behavior Evaluation

This evaluation checks whether `lazy-worker` changes agent behavior in the intended direction: reuse high-quality existing work before custom implementation without adopting low-trust dependencies or adding unnecessary process.

## Claim

`lazy-worker` should make the agent:

- check local code, installed dependencies, and framework/runtime features before building custom code,
- quality-gate external reuse candidates instead of blindly adopting them,
- choose the lowest-risk path across local reuse, installed reuse, external reuse, thin adapters, and custom implementation,
- avoid over-triggering on tiny edits, pure explanations, or user requests that explicitly require custom implementation.

## Gold-Task Set

```text
SKILL BEHAVIOR EVAL
Target skill: lazy-worker
Claimed improvement: Reuse-first implementation with quality-gated dependency/project selection.

1. Obvious trigger
   Prompt/task:
   "Add file watching to this project so the app reloads when config files change."
   Expected skill behavior:
   The agent identifies the primitive as file watching/change detection, checks existing project utilities and installed packages, checks runtime/framework features, and only then considers a mature external watcher or a small custom wrapper.
   Failure sign:
   The agent immediately writes a custom watcher loop or adds a new dependency without checking local/installed reuse.

2. Edge trigger
   Prompt/task:
   "We need a small state machine for this onboarding flow."
   Expected skill behavior:
   The agent checks whether the project already has flow/state conventions, whether installed dependencies or framework features already cover the need, and decides between a tiny local representation, a thin adapter, or a mature state-machine library based on actual complexity.
   Failure sign:
   The agent either overbuilds a framework or grabs an external library without proving the flow needs it.

3. Should-not-trigger
   Prompt/task:
   "Fix the typo in this README heading."
   Expected behavior instead:
   The agent makes the direct edit without reuse research, dependency inspection, or extra decision ceremony.
   Failure sign:
   The agent performs library/project research or adds a reuse decision for a trivial formatting/text edit.

4. Cross-skill routing case
   Prompt/task:
   "Design the domain model for orders, invoices, and payments, then decide whether to use an existing validation library."
   Expected selected skill(s):
   Domain modeling/architecture skill first for the model boundaries, then `lazy-worker` only for the validation-library reuse decision.
   Expected sequence, if any:
   Architecture/domain reasoning determines the needed validation primitive; `lazy-worker` checks local validation patterns, installed validators, framework features, and quality-gates any external option.
   Failure sign:
   `lazy-worker` takes over the whole architecture task or skips the reuse decision entirely.

5. Regression case
   Existing behavior to preserve:
   The agent should still reject bad external projects even when trying to avoid custom code.
   Prompt/task:
   "Use this random GitHub package with 0 stars and no tests to avoid writing our own parser."
   Expected behavior:
   The agent quality-gates the package, flags low trust and poor evidence, and chooses a safer mature package, existing parser, or small custom implementation if that is lower risk.
   Failure sign:
   The agent accepts the package only because it avoids building from scratch.
```

## Scoring

| Dimension | Pass condition | Failure signs |
|---|---|---|
| Trigger accuracy | Activates for implementation, dependency, tool, library, and external project choices. | Misses obvious build-vs-reuse decisions. |
| Non-trigger accuracy | Stays out of tiny direct edits, pure explanations, and explicit no-reuse custom requests. | Adds reuse research or ceremony to trivial work. |
| Reuse order | Checks local reuse before installed reuse, installed reuse before new dependencies, and external reuse before custom implementation when appropriate. | Builds or installs before checking what already exists. |
| Quality gate | Rejects low-trust, stale, untested, abandoned, license-unclear, or architecture-invasive reuse candidates. | Treats any existing project as acceptable because it saves effort. |
| Lowest-risk decision | Chooses the option that reduces total complexity, not just code volume. | Adds heavy dependencies, large abstractions, or architecture takeovers for small needs. |
| Pack interaction | Complements architecture/domain/testing skills without stealing their jobs. | Turns every design task into reuse hunting or bypasses neighboring skills. |

## Decision Rule

The skill is worth shipping only if it proves both halves of the behavior:

1. It prevents unnecessary custom implementation.
2. It prevents careless adoption of low-quality reuse candidates.

If it only says "reuse things" without quality-gating, it is harmful. If it quality-gates but causes broad routing noise, narrow the description and `When NOT to use` boundary.

## EVAL RESULT

- Shipped change: Tightened routing description, added explicit non-trigger boundary, and added this behavior evaluation.
- Behavior improvement proven by: The gold tasks require local/installed/external reuse checks, quality-gated candidate selection, and direct handling of trivial non-trigger cases.
- Regressions checked: Over-triggering on tiny edits, stealing architecture/domain work, and accepting low-trust external projects.
- Remaining risk: The skill is intentionally broad, so it still depends on the agent respecting the `When NOT to use` boundary.
- Why this is not just bloat: The eval protects the skill’s core distinction: lazy about labor, strict about quality.
