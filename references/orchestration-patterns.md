# Orchestration Patterns

Reference catalog for using subagents without turning every workflow into expensive coordination theater. Use this file when deciding whether doubt-driven review needs one fresh-context reviewer, several independent reviewers, or a team-style investigation.

Governing rule: **the top-level session, user, or explicit command is the orchestrator. Reviewer personas report findings; they do not invoke other personas.** Skills are workflow guidance inside an agent, not an excuse for nested agent trees.

## Endorsed Patterns

### 1. Direct invocation

Single agent, single perspective, single artifact.

```text
user → reviewer → report → user
```

Use when the task is one perspective on one artifact and can be described in one sentence.

Examples:

- Review this diff for maintainability problems.
- Find security issues in this auth function.
- Identify missing tests for this checkout flow.

Cost: one agent context. This is the baseline every orchestrated pattern must beat.

### 2. Saved single-agent command

A command, prompt, or IDE action wraps one persona with a repeatable setup.

```text
/review → reviewer + relevant skill → report
```

Use when the same single-agent workflow happens repeatedly with the same setup.

Anti-signal: if the command mostly decides which persona to call, delete the command and call the persona directly.

### 3. Parallel fan-out with merge

Multiple subagents inspect the same input independently. The top-level session merges their reports into one decision.

```text
                 ┌─→ code reviewer     ─┐
fan out input ───┼─→ security reviewer ─┤→ merge → decision + actions
                 └─→ test reviewer     ─┘
```

Use when all of these are true:

- The subtasks are independent.
- Each reviewer produces a different kind of finding.
- Each reviewer benefits from its own context window.
- The merge step is small enough for the top-level session.
- The expected improvement justifies the extra model calls.

Validation before using this pattern:

- [ ] Can all reviewers run without ordering dependencies?
- [ ] Are their lenses genuinely different, not duplicated?
- [ ] Can the top-level session reconcile conflicts without another large investigation?
- [ ] Does parallelism or context isolation materially improve the result?

If any answer is no, use direct invocation.

### 4. Sequential user-driven pipeline

The user or explicit command sequence carries context between dependent phases.

```text
user runs: define → plan → build → test → review → ship
```

Use when each step depends on the previous output and human judgment between steps adds value.

Do not hide this inside a lifecycle orchestrator by default. Automated lifecycle agents tend to summarize away nuance, skip useful checkpoints, and double token cost through handoff paraphrasing.

### 5. Research isolation

A read-only research subagent consumes a large body of code/docs and returns a compact digest.

```text
top-level session → research subagent reads broad context → digest → top-level session continues
```

Use when:

- The main session needs to stay focused on a downstream task.
- The investigation result is much smaller than the source material.
- The decision benefits from preserving main-context room.

Examples:

- Find every call site of a deprecated API.
- Summarize the relevant ADRs for a caching decision.
- Map all modules touched by a proposed boundary change.

Use your IDE's built-in read-only exploration subagent if it has one. Otherwise use a generic research subagent with write/edit tools disabled when possible.

### 6. Team-style competing-hypothesis investigation

Several agents can see each other's hypotheses and actively try to disprove them.

```text
team investigates → agents challenge competing theories → consensus root cause
```

Use this instead of parallel fan-out when the goal is to discover the correct explanation among plausible competing theories, not to get independent verdicts on a known artifact.

Good fit:

- Intermittent production bug with several plausible root causes.
- Architectural mystery where multiple explanations fit the evidence.
- Failure mode where one agent is likely to anchor on the first plausible theory.

Poor fit:

- Known diff needs review.
- Known artifact needs go/no-go.
- The agents do not need to see each other's reasoning.

Team-style investigation is more expensive than subagent fan-out. Use it only when adversarial interaction between investigators is the source of value.

## Anti-Patterns

### A. Persona calls persona

A reviewer decides to spawn another reviewer.

Problem: responsibility becomes unclear, context balloons, and nobody owns the final decision.

Fix: reviewers report findings to the top-level session. The top-level session decides whether to spawn more reviewers.

### B. Deep agent trees

```text
top-level → reviewer → reviewer → reviewer
```

Problem: cost and distortion compound while accountability decreases.

Fix: keep orchestration one level deep unless the user explicitly requests a deeper investigation and the IDE safely supports it.

### C. Duplicate lenses

Multiple agents review the same artifact with the same prompt.

Problem: this usually produces redundant confidence, not better evidence.

Fix: use one reviewer, or give each reviewer a genuinely different contract.

### D. Big merge after big fan-out

The top-level session receives several huge reports and cannot reconcile them cleanly.

Problem: the merge becomes another unreviewed artifact.

Fix: constrain report shape, split the artifact, or use sequential review.

### E. Orchestration for ceremony

A command spawns agents because orchestration feels sophisticated.

Problem: cost rises while output quality does not.

Fix: compare against direct invocation first. Use the cheapest pattern that changes the result.

## Doubt-Driven Defaults

For `doubt-driven-development`, default to Pattern 1 or Pattern 5:

- One fresh-context adversarial reviewer for a small artifact.
- One read-only research subagent when the contract depends on broad codebase context.

Escalate to Pattern 3 only when different reviewer lenses would produce materially different findings, such as security, data integrity, and testability on the same risky artifact.

Escalate to Pattern 6 only when the problem is a competing-hypothesis investigation rather than review of a known artifact.

## Portable IDE Guidance

Map the patterns onto whatever your IDE supports:

- Named subagents: choose the closest read-only or specialist role.
- Generic subagents: paste the role, artifact, contract, and output format explicitly.
- Team/collaborative mode: reserve it for competing-hypothesis investigations.
- No subagents: use the degraded self-review fallback only after stating that the result is not fresh-context review.

Do not assume one vendor's agent names, frontmatter fields, command syntax, environment variables, or plugin layout. Those are adapter details, not the pattern.
