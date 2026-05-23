# Skill Template Reference

Use these templates as starting points, not compliance targets. Pick the smallest shape that makes skill activation, execution, and verification reliable. Do not add sections that do not change agent behavior, routing, safety, or evidence quality. Replace placeholder filenames before use; do not copy placeholder links into a final skill.

## Reusable Workflow Skill Template

Use this for skills that guide agents through a repeatable process.

````md
---
name: lowercase-hyphen-name
description: Guides agents through [task]. Use when [specific trigger conditions], including [secondary trigger conditions if useful].
---

# Human Readable Skill Name

## Overview

[One to three paragraphs explaining the skill's operating principle. State the core discipline directly. Explain what the skill prevents and what good execution looks like.]

## When to Use

- [Specific trigger]
- [Specific trigger]
- [Specific trigger]
- [Risk condition or smell that should trigger the skill]

**When NOT to use:** [Clear exclusion. Explain the common overuse case and what to do instead.]

## The [Task] Workflow

```
1. STEP NAME → Concrete purpose
2. STEP NAME → Concrete purpose
3. STEP NAME → Concrete purpose
4. VERIFY   → Evidence that proves completion
```

### Step 1: [Verb]

[What the agent must do. Include decision criteria, checkpoints, or required questions.]

### Step 2: [Verb]

[What the agent must do next. Prefer concrete actions over prose explanation.]

### Step 3: [Verb]

[Continue the workflow. Add tables or small examples only when they change agent behavior.]

## [Optional Domain-Specific Section]

[Use only when the skill needs a compact table, decision tree, checklist, budget, matrix, or examples that belong in the main path.]

## See Also

[Remove this section unless there is a real supporting file.]

For [advanced topic / long checklist / examples], see a real supporting file only after creating that file and linking its actual name.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "[Excuse agents use to skip the process]" | [Why that excuse is wrong and what to do instead.] |
| "[Excuse agents use to overbuild]" | [Why that excuse is wrong and what simpler path to use.] |
| "[Excuse agents use to skip verification]" | [Why evidence is required.] |

## Red Flags

- [Observable sign the skill is being misapplied]
- [Observable sign the agent is skipping a required step]
- [Observable sign the output is too vague]
- [Observable sign verification is missing]

## Verification

After using this skill:

- [ ] [Concrete artifact, decision, command, test, or evidence exists]
- [ ] [The triggering problem was addressed]
- [ ] [Rejected alternatives or tradeoffs are documented when relevant]
- [ ] [Supporting files are linked only when needed]
- [ ] [No unverifiable claims remain]
````

## Router or Meta-Skill Skeleton

Use this for skills that choose between other skills or define pack-level operating rules.

```md
---
name: lowercase-hyphen-name
description: Routes agents across [skill family / decision surface]. Use when [selection, orchestration, or pack-level trigger].
---

# Human Readable Skill Name

## Purpose

[What this router decides and what it must not decide.]

## Routing Rules

| User Need / Signal | Use | Do Not Use |
|---|---|---|
| [Trigger] | `[skill-name]` | [Boundary] |

## Selection Rules

- Prefer the smallest applicable skill.
- Do not load a skill just because it is adjacent.
- If no installed skill applies, proceed normally.

## Verification

- [ ] The selected skill matches the actual user request.
- [ ] No absent or unrelated skill was invoked.
- [ ] The router did not add process overhead beyond selection.
```

## Reference-Heavy Helper Skeleton

Use this for compact judgment support where the main job is consultation, not a linear process.

```md
---
name: lowercase-hyphen-name
description: Provides reference guidance for [domain]. Use when [specific lookup, classification, or judgment support trigger].
---

# Human Readable Skill Name

## Scope

[What this reference helps decide or check.]

## When to Consult

- [Specific trigger]
- [Specific trigger]

**When NOT to use:** [Exclusion or overuse case.]

## Usage Rules

- [Rule that changes agent behavior]
- [Rule that prevents misuse]

## Reference Map

| Need | See |
|---|---|
| [Need] | Actual supporting filename, after that file exists |

## Verification

- [ ] The consulted reference directly affected a decision, artifact, or risk check.
- [ ] The answer did not treat background material as mandatory procedure.
```

## Utility or Script Skill Skeleton

Use this for deterministic helpers with clear inputs and outputs.

```md
---
name: lowercase-hyphen-name
description: Runs or guides [utility]. Use when [input/output trigger or deterministic operation].
---

# Human Readable Skill Name

## Inputs

- [Required input]
- [Optional input]

## Invocation Rules

- [When to run]
- [Safety limit]
- [What not to modify]

## Outputs

- [Expected output]
- [Failure mode]

## Verification

- [ ] Inputs were valid.
- [ ] The expected output exists.
- [ ] Failures were reported without guessing.
```

## Diagnostic Checklist Skeleton

Use this when checks can be applied in parallel or selectively instead of as a strict sequence.

```md
---
name: lowercase-hyphen-name
description: Diagnoses [problem class]. Use when [symptoms, failure modes, or uncertainty triggers].
---

# Human Readable Skill Name

## Symptoms

- [Observable symptom]
- [Observable symptom]

**When NOT to use:** [Exclusion or overuse case, if relevant.]

## Checks

| Check | Evidence | Escalate When |
|---|---|---|
| [Check] | [Artifact, command, trace, log, or observation] | [Threshold] |

## Red Flags

- [Observable sign of misdiagnosis]
- [Observable sign evidence is missing]

## Verification

- [ ] Each claim is tied to evidence.
- [ ] Unknowns are marked as unknown instead of inferred.
- [ ] The next action follows from the diagnosis.
```
