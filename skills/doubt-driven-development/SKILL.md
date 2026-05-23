---
name: doubt-driven-development
description: Uses fresh-context adversarial review for costly, uncertain, boundary-crossing, or weakly-evidenced decisions. Use when a wrong answer would be expensive, confidence is high but evidence is thin, or the artifact is hard to validate directly.
---

# Doubt-Driven Development

## Overview

A confident answer is not a correct one. Long sessions accumulate context that quietly turns assumptions into "facts" without anyone noticing. Doubt-driven development is the discipline of materializing a fresh-context reviewer — biased to **disprove**, not approve — before high-cost or weakly-evidenced outputs stand.

This is not `/review`. `/review` is a verdict on a finished artifact. This is an in-flight posture: consequential decisions get cross-examined while course-correction is still cheap.

## When to Use

Use this skill when at least one of these is true:

- A wrong answer would be costly to debug, reverse, deploy, migrate, or explain later
- The decision crosses a module, service, security, data, public API, deployment, or ownership boundary
- The artifact is hard to test directly, or tests cannot fully prove the relevant claim
- Confidence is high but evidence is weak, implicit, or mostly based on session momentum
- The work is in unfamiliar code and the existing invariants are not fully understood
- The claim depends on properties the type system or compiler cannot verify, such as thread safety, idempotence, ordering, invariants, isolation, or data integrity
- The blast radius is irreversible or high-stakes, such as production deploys, data migrations, security-sensitive logic, payment/money movement, or public API behavior

A decision is usually **not worth a doubt cycle** merely because it is technically non-trivial. It must also have meaningful cost, uncertainty, boundary risk, weak evidence, or limited direct testability.

**When NOT to use:**

- Mechanical operations such as renaming, formatting, file moves, import cleanup, or generated-code refreshes
- Following a clear, unambiguous user instruction where the risk is low
- Reading or summarizing existing code
- One-line changes with obvious correctness
- Routine implementation choices already covered by tests and local conventions
- Pure tooling operations such as running tests or listing files
- The user has explicitly asked for speed over verification

If you doubt every keystroke, you ship nothing. The skill applies only when the cost of being wrong justifies the review overhead.

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
- Assertion: the claim plus the evidence that supposedly supports it, kept distinct from the Step 1 CLAIM block, which is the orchestrator's hypothesis under scrutiny

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

Prefer a second subagent pass when the first pass found substantive issues, the fix changed the artifact materially, or the remaining risk is still expensive, boundary-crossing, security/data-sensitive, or hard to test directly.

Do not run a second pass by ritual. Skip it when the first pass produced only trivial findings, the artifact became obvious after the fix, the change is fully covered by direct tests, or subagent orchestration is unavailable/unsafe.

When used, the second pass should be isolated from both the orchestrator reasoning and the first reviewer’s findings unless you are explicitly asking it to audit those findings.

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
4. Do not recursively spawn reviewers. After 3 total doubt cycles, escalate to the user or decompose the artifact.
5. In non-interactive/autonomous contexts, use judgment: prefer a second-pass subagent for expensive unresolved risk, and skip it for trivial or directly proven artifacts. Announce whether it was used or skipped and why when reporting the doubt cycle.

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
| "I'm confident, skip the doubt step" | Confidence is least trustworthy when evidence is thin, stakes are high, or the session has been reinforcing the same assumption for a long time. |
| "Spawning a reviewer is expensive" | It is expensive, so use it where the cost of being wrong is higher than the review overhead. |
| "The reviewer will just nitpick" | Only if unscoped. Constrain the prompt to issues that would make the artifact fail under the contract. |
| "I'll do doubt at the end with `/review`" | `/review` is a final gate. Doubt-driven catches wrong directions early when course-correction is cheap. |
| "If I doubt every step I'll never ship" | Correct. The skill applies to costly, uncertain, boundary-crossing, or weakly-evidenced decisions, not every technically non-trivial choice. |
| "Two opinions are always better than one" | Not when the second has less context and produces noise. Reconcile, don't defer. |
| "The reviewer disagreed so I was wrong" | The reviewer lacks your context — disagreement is information, not verdict. Re-read the artifact, classify, then decide. |
| "A second reviewer is always required" | No. Use a second pass when it is likely to catch a different costly failure mode. Skip it when it would only add ritual. |

## Red Flags

- Spawning a fresh-context reviewer for a one-line rename, formatting change, or routine local edit
- Treating reviewer output as authoritative without re-reading the artifact text
- Looping >3 cycles without escalating to the user
- Prompting the reviewer with "is this good?" instead of "find issues"
- Skipping doubt under time pressure on a high-stakes decision
- Re-spawning fresh-context on an unchanged artifact merely for reassurance
- **Doubt theater (checkable signal)**: across 2 or more cycles where the reviewer surfaced substantive findings, zero findings were classified as actionable. You are validating, not doubting. Stop and escalate.
- Doubting only after committing — that's `/review`, not doubt-driven development
- Running the second-pass subagent by default after the artifact is already trivial or directly proven
- Falling back silently when fresh-context subagents are unavailable — surface the limitation and mark the review degraded
- Stripping the contract from the reviewer's input
- Passing the CLAIM to the reviewer, which biases toward agreement

## Interaction with Other Skills

- **`source-driven-development`**: SDD verifies *facts about frameworks* against official docs. Doubt-driven verifies *your reasoning about the artifact*. SDD checks the API exists; doubt-driven checks you used it correctly under the contract.
- **`test-driven-development`**: TDD's RED step is doubt made concrete — a failing test is a disproof attempt. When TDD applies, that failing test *is* the doubt step for behavioral claims.

## Verification

After applying doubt-driven development:

- [ ] The decision justified a doubt cycle because the cost, uncertainty, boundary risk, weak evidence, or limited direct testability was explicit
- [ ] The claim was named explicitly before standing
- [ ] At least one fresh-context review was used for the selected artifact, unless a TDD RED test already provided concrete disproof for a behavioral claim
- [ ] The reviewer received ARTIFACT + CONTRACT — NOT the CLAIM, NOT your reasoning
- [ ] The reviewer's prompt was adversarial ("find issues"), not validating ("is it good")
- [ ] Findings were classified against the artifact text, not rubber-stamped, using the precedence: contract misread / actionable / trade-off / noise
- [ ] A stop condition was met: trivial findings, 3 cycles, or user override
- [ ] A second-pass subagent was used only when remaining risk justified it, or skipped because the artifact was trivial, directly proven, unavailable, or unsafe to review that way
- [ ] If no safe subagent mechanism was available, the output marked the review degraded rather than pretending it was fresh-context
