---
name: doubt-driven-development
description: Subjects every non-trivial decision to a fresh-context adversarial review before it stands. Use when correctness matters more than speed, when working in unfamiliar code, when stakes are high (production, security-sensitive logic, irreversible operations), or any time a confident output would be cheaper to verify now than to debug later.
---

# Doubt-Driven Development

## Overview

A confident answer is not a correct one. Long sessions accumulate context that quietly turns assumptions into "facts" without anyone noticing. Doubt-driven development is the discipline of materializing a fresh-context reviewer — biased to **disprove**, not approve — before any non-trivial output stands.

This is not `/review`. `/review` is a verdict on a finished artifact. This is an in-flight posture: non-trivial decisions get cross-examined while course-correction is still cheap.

## When to Use

A decision is **non-trivial** when at least one of these is true:

- It introduces or modifies branching logic
- It crosses a module or service boundary
- It asserts a property the type system or compiler cannot verify (thread safety, idempotence, ordering, invariants)
- Its correctness depends on context the future reader cannot see
- Its blast radius is irreversible (production deploy, data migration, public API change)

Apply the skill when:

- About to make an architectural decision under uncertainty
- About to commit non-trivial code
- About to claim a non-obvious fact ("this is safe", "this scales", "this matches the spec")
- Working in code you don't fully understand

**When NOT to use:**

- Mechanical operations (renaming, formatting, file moves)
- Following a clear, unambiguous user instruction
- Reading or summarizing existing code
- One-line changes with obvious correctness
- Pure tooling operations (running tests, listing files)
- The user has explicitly asked for speed over verification

If you doubt every keystroke, you ship nothing. The skill applies only to non-trivial decisions as defined above.

## Loading Constraints

This skill is designed for a **main-session orchestrator** or other top-level IDE workflow where Step 3 (DOUBT, detailed below) can spawn a fresh-context reviewer. Use [`orchestration-patterns.md`](../../references/orchestration-patterns.md) only when choosing between one reviewer, research isolation, parallel fan-out, or team-style investigation; ordinary doubt cycles should stay lightweight.

- **Do not attach this skill to an automatically-invoked reviewer/persona that will recursively spawn more reviewers.** Doubt-driven review should be orchestrated from the top-level session unless your IDE explicitly supports safe nested subagents.
- **If you find yourself applying this skill from inside a subagent context** and nested subagents are not available or not safe, surface that limitation and let the main session run the doubt cycle. As a last resort only, use a degraded self-questioning fallback: rewrite ARTIFACT + CONTRACT as a fresh self-prompt with a hard mental separator from your prior reasoning, and walk Steps 1–5. This is **not fresh-context review** because you carry your own context, so flag the result as degraded and prefer escalation whenever the user is reachable.

## Subagent Orchestration Rules

These are the load-bearing local rules for this skill. The broader pattern catalog lives in [`orchestration-patterns.md`](../../references/orchestration-patterns.md), but do not load it for simple one-reviewer cycles unless orchestration choice matters.

- Run the doubt cycle from the top-level session or orchestrator, not from an automatically invoked reviewer.
- Use a fresh-context or reduced-context subagent whose only job is to find issues in ARTIFACT + CONTRACT.
- Do not let reviewer subagents recursively spawn more reviewers unless the IDE explicitly supports safe nested subagents and the user has asked for that depth.
- Reviewer subagents should be read-only by default. They inspect and report; the orchestrator reconciles and edits.
- Prefer one reviewer per reviewable unit. If the artifact is too large, split it instead of asking one reviewer to evaluate everything.
- Never pass the orchestrator's CLAIM, private reasoning, or desired conclusion to the reviewer.
- Treat reviewer output as findings, not authority. The orchestrator owns classification and final decisions.

## The Process

Copy this checklist when applying the skill:

```
Doubt cycle:
- [ ] Step 1: CLAIM — wrote the claim + why-it-matters
- [ ] Step 2: EXTRACT — isolated artifact + contract, stripped reasoning
- [ ] Step 3: DOUBT — invoked fresh-context reviewer with adversarial prompt
- [ ] Step 4: RECONCILE — classified every finding against the artifact text
- [ ] Step 5: STOP — met stop condition (trivial findings, 3 cycles, or user override)
```

### Step 1: CLAIM — Surface what stands

Name the decision in two or three lines:

```
CLAIM: "The new caching layer is thread-safe under the
        read-heavy workload described in the spec."
WHY THIS MATTERS: a race here corrupts user data and is
                  hard to detect in QA.
```

If you can't write the claim that compactly, you have a vibe, not a decision. Surface it before scrutinizing it.

### Step 2: EXTRACT — Smallest reviewable unit

A fresh-context reviewer needs the **artifact** and the **contract**, not the journey.

- Code: the diff or the function — not the whole file
- Decision: the proposal in 3–5 sentences plus the constraints it has to satisfy
- Assertion: the claim plus the evidence that supposedly supports it (kept distinct from the Step 1 CLAIM block, which is the orchestrator's hypothesis under scrutiny)

Strip your reasoning. If you hand over conclusions, you'll get back validation of your conclusions. The unit must be small enough that a reviewer can hold it in mind in one read — if it's a 500-line PR, decompose first.

### Step 3: DOUBT — Invoke the fresh-context reviewer

The reviewer's prompt **must be adversarial**. Framing decides the answer.

```
Adversarial review. Find what is wrong with this artifact.
Assume the author is overconfident. Look for:
- Unstated assumptions
- Edge cases not handled
- Hidden coupling or shared state
- Ways the contract could be violated
- Existing conventions this might break
- Failure modes under unexpected input

Do NOT validate. Do NOT summarize. Find issues, or state
explicitly that you cannot find any after thorough examination.

ARTIFACT: <paste artifact>
CONTRACT: <paste contract>
```

**Pass ARTIFACT + CONTRACT only. Do NOT pass the CLAIM.** Handing the reviewer your conclusion biases it toward agreement. The reviewer must independently determine whether the artifact satisfies the contract.

Use the IDE's subagent mechanism to create an isolated or reduced-context reviewer. Prefer a role-specific reviewer when your IDE has one that matches the artifact; otherwise launch a generic fresh-context subagent with the adversarial prompt.

**The adversarial prompt above takes precedence over the reviewer persona's default response shape.** Many code-review personas produce balanced verdicts with both strengths and weaknesses; doubt-driven needs issues-only output. Paste the adversarial prompt verbatim into the invocation so it overrides the persona's default. If a persona's response shape can't be overridden cleanly, fall back to a generic subagent with the adversarial prompt.

#### Preferred second-pass subagent review

**prefer a second subagent pass by default** after the first review/reconciliation. The second pass is not an exceptional escalation; it is the normal way to get a second independent failure search before the artifact stands. The second pass should be isolated from both the orchestrator reasoning and the first reviewer’s findings unless you are explicitly asking it to audit those findings.

A second subagent is especially important when at least one of these is true:

- The first reviewer found substantive issues and the fix changed the artifact materially
- The artifact touches production safety, security, data migration, public API behavior, money movement, or irreversible user data
- The first reviewer lacked domain context and the risk still feels unresolved
- The artifact spans multiple seams and one reviewer would likely miss a boundary-specific failure mode

Prefer a different review angle, not a duplicate pass:

- **Correctness reviewer:** invariants, edge cases, data integrity, algorithmic failure modes
- **Integration reviewer:** boundaries, conventions, dependency direction, config/deploy coupling
- **Security reviewer:** trust boundaries, injection, authz/authn, secret handling, unsafe defaults
- **Test reviewer:** missing proof, weak assertions, untested branches, fixture realism

Second-pass prompt shape:

```
Adversarial second-pass review. Find issues the first reviewer may have missed.
Do not summarize. Do not validate. Focus on: <chosen review angle>.

ARTIFACT: <paste current artifact>
CONTRACT: <paste contract>
```

Rules:

1. Keep the second subagent read-only. It reports; the orchestrator edits.
2. Pass the current ARTIFACT + CONTRACT only. Do not pass the CLAIM, private reasoning, or desired outcome.
3. Do not show the first reviewer's findings unless the second subagent is explicitly auditing the reconciliation. Independent review catches different mistakes; finding-audit checks whether the first findings were handled correctly. Do not mix those modes.
4. Do not recursively spawn reviewers. Two subagent passes are the preferred default; after 3 total doubt cycles, escalate to the user or decompose the artifact.
5. In non-interactive/autonomous contexts, still prefer exactly one second-pass subagent after the first reconciliation. Skip it only when it is impossible, unsafe, or clearly wasteful because the artifact has become trivial. Announce whether it was used or skipped and why.

If no safe subagent mechanism is available, state that fresh-context review could not be performed and use the degraded self-questioning fallback from Loading Constraints. Do not replace the subagent path with a separate external-tool workflow by default.

### Step 4: RECONCILE — Fold findings back

The reviewer's output is data, not verdict. **You are still the orchestrator.** Re-read the artifact text against each finding before classifying — rubber-stamping the reviewer is the same failure mode as ignoring it.

For each finding, classify in this **precedence order** (first matching class wins):

1. **Contract misread** — reviewer flagged something specifically because the CONTRACT you provided was unclear or incomplete. Fix the contract first, re-classify on the next cycle.
2. **Valid + actionable** — real issue requiring a change to the artifact. Change it, re-loop.
3. **Valid trade-off** — issue is real but cost of fixing exceeds cost of accepting. Document the trade-off explicitly so the user sees it.
4. **Noise** — reviewer flagged something that's actually correct under context the reviewer didn't have. Note it, move on, and ask: would adding that context to the contract have prevented the false flag?

A fresh reviewer can be wrong because it lacks context. Don't defer just because it's "fresh."

### Step 5: STOP — Bounded loop, not recursion

Stop when:

- Next iteration returns only trivial or already-considered findings, **or**
- 3 cycles completed (escalate to user, don't grind a fourth alone), **or**
- User explicitly says "ship it"

If after 3 cycles the reviewer still surfaces substantive issues, the artifact may not be ready. Surface this to the user — three unresolved cycles is information about the artifact, not a reason to keep looping.

If 3 cycles is "obviously insufficient" because the artifact is large: the artifact is too big — return to Step 2 and decompose. Do not lift the bound.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'm confident, skip the doubt step" | Confidence correlates poorly with correctness on novel problems. Moments of certainty are exactly when blind spots hide. |
| "Spawning a reviewer is expensive" | Debugging a wrong commit in production is more expensive. The check is bounded; the bug isn't. |
| "The reviewer will just nitpick" | Only if unscoped. Constrain the prompt to "issues that would make this fail under the contract." |
| "I'll do doubt at the end with `/review`" | `/review` is a final gate. Doubt-driven catches wrong directions early when course-correction is cheap. By PR time it's too late. |
| "If I doubt every step I'll never ship" | The skill applies to non-trivial decisions, not every keystroke. Re-read "When NOT to Use." |
| "Two opinions are always better than one" | Not when the second has less context and produces noise. Reconcile, don't defer. |
| "The reviewer disagreed so I was wrong" | The reviewer lacks your context — disagreement is information, not verdict. Re-read the artifact, classify, then decide. |
| "A second reviewer is overhead" | For this skill, the work is already non-trivial/hard enough to justify doubt. Prefer one second-pass subagent unless it is impossible, unsafe, or the artifact has become trivial. |

## Red Flags

- Spawning a fresh-context reviewer for a one-line rename or formatting change
- Treating reviewer output as authoritative without re-reading the artifact text
- Looping >3 cycles without escalating to the user
- Prompting the reviewer with "is this good?" instead of "find issues"
- Skipping doubt under time pressure on a high-stakes decision
- Re-spawning fresh-context on an unchanged artifact (you'll get the same findings; you're stalling)
- **Doubt theater (checkable signal)**: across 2 or more cycles where the reviewer surfaced substantive findings, zero findings were classified as actionable. You are validating, not doubting. Stop and escalate.
- Doubting only after committing — that's `/review`, not doubt-driven development
- Skipping the preferred second-pass subagent without a concrete reason such as unavailable subagents, unsafe nested orchestration, or a now-trivial artifact
- Falling back silently when fresh-context subagents are unavailable — surface the limitation and mark the review degraded
- Stripping the contract from the reviewer's input
- Passing the CLAIM to the reviewer (biases toward agreement)

## Interaction with Other Skills

- **`source-driven-development`**: SDD verifies *facts about frameworks* against official docs. Doubt-driven verifies *your reasoning about the artifact*. SDD checks the API exists; doubt-driven checks you used it correctly under the contract.
- **`test-driven-development`**: TDD's RED step is doubt made concrete — a failing test is a disproof attempt. When TDD applies, that failing test *is* the doubt step for behavioral claims.

## Verification

After applying doubt-driven development:

- [ ] Every non-trivial decision (per the definition above) was named explicitly as a CLAIM before standing
- [ ] At least one fresh-context review per non-trivial artifact (a failing test produced by TDD's RED step satisfies this for behavioral claims, per Interaction with Other Skills)
- [ ] The reviewer received ARTIFACT + CONTRACT — NOT the CLAIM, NOT your reasoning
- [ ] The reviewer's prompt was adversarial ("find issues"), not validating ("is it good")
- [ ] Findings were classified against the artifact text (not rubber-stamped) using the precedence: contract misread / actionable / trade-off / noise
- [ ] A stop condition was met (trivial findings, 3 cycles, or user override)
- [ ] A second-pass subagent was used by default, or skipped only because it was impossible, unsafe, or the artifact had become trivial
- [ ] If no safe subagent mechanism was available, the output marked the review degraded rather than pretending it was fresh-context
