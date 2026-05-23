---
name: handoff
description: Creates compact continuity handoffs for another agent or session. Use when the user needs the current conversation, decisions, evidence, changed artifacts, known gaps, and suggested next skills packaged so work can continue without losing context.
argument-hint: "What will the next session be used for?"
---

# Handoff

## Overview

Create a handoff document that lets a fresh agent continue the work without rediscovering decisions, retesting already-checked claims, or losing known risks. A good handoff preserves continuity; it does not duplicate full artifacts that already exist elsewhere.

Save the handoff to the temporary directory of the user's OS, not the current workspace, unless the user explicitly asks for a different location.

## When to Use

Use when:

- the user asks to prepare a handoff, compact the session, summarize for another agent/session, or continue later,
- the conversation contains decisions, changed files, commands, tests, or unresolved risks that another agent must preserve,
- work is being transferred between tools, agents, or sessions,
- the next session has a specific focus and needs a starting packet.

**When NOT to use:** Do not use for a normal short summary, status reply, or artifact that already has its own plan/ADR/PRD/issue. Reference existing artifacts instead of duplicating them.

## Process

### 1. Identify the next-session purpose

If the user passed arguments, treat them as the next session's focus. Otherwise infer the continuation target from the current conversation.

```text
NEXT SESSION PURPOSE
- Continue implementation
- Review/audit
- Debug/diagnose
- Package/release
- Revise skill/library
- Other: ...
```

### 2. Preserve decisions and evidence

Capture what the next agent must trust or re-check.

Include:

- user goal and constraints,
- decisions made and why,
- files/artifacts created or changed,
- commands/tests run and results,
- evidence gathered from docs, files, outputs, or inspections,
- rejected options when they prevent repeated bad work,
- unresolved questions or known gaps.

Do not claim evidence exists unless it was actually produced.

### 3. Preserve reliability context

For engineering or skill-library work, include reliability-critical continuity:

```text
RELIABILITY CONTINUITY
- Invariants / acceptance examples:
- Failure modes considered:
- Test seams or commands:
- Known untested paths:
- Stop condition for the next agent:
```

Omit fields that do not apply. Do not invent reliability risks just to fill the section.

### 4. Suggest next skills

Include a `Suggested Skills` section using only skills present in the pack. Recommend the shortest useful sequence and explain the trigger in one line.

Example:

```text
Suggested Skills
- shift-left-testing — test seams and quality gates are still unclear.
- test-driven-development — first behavior is ready for red-green-refactor.
```

### 5. Redact sensitive information

Redact API keys, passwords, tokens, secrets, precise credentials, private personal data, and unnecessary identifiers. Keep references to paths, issue IDs, or artifact names when they are needed for continuity.

## Output Format

```text
# Handoff

## Purpose
- Next-session focus:
- User goal:
- Constraints/preferences:

## Current State
- What is done:
- What changed:
- Important files/artifacts:

## Decisions Made
1. Decision:
   Why:
   Evidence:

## Evidence and Verification
- Commands/tests run:
- Results:
- Inspections performed:
- Claims not yet verified:

## Reliability Continuity, If Relevant
- Invariants / acceptance examples:
- Failure modes considered:
- Test seams or commands:
- Known untested paths:
- Stop condition:

## Open Questions / Known Gaps
- ...

## Suggested Skills
- ...

## Sensitive Data Handling
- Redactions made:
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "A general summary is enough." | Continuity fails when decisions, evidence, gaps, and stop conditions are missing. |
| "I'll paste the whole plan again." | Duplicate artifacts go stale. Link or reference existing artifacts instead. |
| "The next agent can rerun everything." | Rerunning without context wastes time and can repeat already-rejected choices. |
| "Known gaps make the work look unfinished." | Known gaps are exactly what the next agent needs to avoid false confidence. |
| "Suggested skills are obvious." | The next agent may not know this pack; name the shortest useful sequence. |

## Red Flags

- handoff duplicates a full PRD, ADR, plan, diff, or issue instead of linking it,
- no commands/tests/results are listed after engineering work,
- decisions are listed without reasons or evidence,
- known gaps are omitted,
- suggested skills include skills not present in the pack,
- sensitive data is copied verbatim,
- the next action is vague,
- stop condition is missing for complex continuation work.

## Verification

After writing the handoff:

- [ ] It states the next-session purpose
- [ ] It includes user goal, constraints, and current state
- [ ] Decisions include reasons and evidence where available
- [ ] Commands/tests/inspections are recorded with results or clearly marked unrun
- [ ] Known gaps and open questions are explicit
- [ ] Reliability continuity is included for engineering or skill-library work when relevant
- [ ] Suggested skills are from this pack only
- [ ] Existing artifacts are referenced instead of duplicated
- [ ] Sensitive information is redacted
- [ ] The next action and stop condition are clear
