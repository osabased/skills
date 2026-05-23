# Skill Behavior Evaluation

Use this only for meaningful skill creation or revision. It is a lightweight behavior check, not a full benchmark suite. The goal is to prove that the skill changes agent behavior in the intended direction without causing routing confusion, extra ceremony, or regression in neighboring skills.

## When to Use

Use this file when:

- creating a new skill,
- revising a skill's trigger conditions, workflow, verification, support files, or routing behavior,
- adding a new skill to a pack,
- changing a router/meta-skill,
- claiming that a skill improves quality, reliability, or agent output.

Skip this file for tiny typo fixes, formatting-only edits, broken-link repair, or changes that cannot affect when the skill fires or what the agent does.

## Gold-Task Set

Create the smallest task set that would catch a bad version of the skill. Three cases are enough for a tiny change; five are appropriate for a new or behavior-changing skill.

```text
SKILL BEHAVIOR EVAL
Target skill:
Claimed improvement:

1. Obvious trigger
   Prompt/task:
   Expected skill behavior:
   Failure sign:

2. Edge trigger
   Prompt/task:
   Expected skill behavior:
   Failure sign:

3. Should-not-trigger
   Prompt/task:
   Expected behavior instead:
   Failure sign:

4. Cross-skill routing case
   Prompt/task:
   Expected selected skill(s):
   Expected sequence, if any:
   Failure sign:

5. Regression case
   Existing behavior to preserve:
   Prompt/task:
   Expected behavior:
   Failure sign:
```

## Scoring

Score only what matters for the change. Use pass/fail unless a graded score would change the decision.

| Dimension | Pass condition | Failure signs |
|---|---|---|
| Trigger accuracy | The skill activates for the right requests. | Broad description, vague trigger, missed obvious use case. |
| Non-trigger accuracy | The skill stays out of unrelated work. | It fires for generic tasks or steals another skill's job. |
| Output shape | The skill produces a concrete artifact, decision, command, or evidence. | The output is essay-like, generic, or hard to verify. |
| Specificity | Guidance changes what the agent does. | Advice is obvious, redundant, or already covered elsewhere. |
| Overhead | Added process is proportional to the task. | The skill makes simple tasks slower or more ceremonial. |
| Pack interaction | Routing and support-file links stay coherent. | Router ambiguity, broken references, duplicated responsibilities. |

## Decision Rule

A skill change is worth shipping only if the eval shows at least one concrete behavior improvement and no material routing regression.

If the eval is weak, prefer one of these smaller moves:

- patch an existing skill instead of adding a new one,
- move reference material into a support file,
- narrow the description,
- add one verification item instead of a new workflow,
- do nothing.

## Reporting Format

```text
EVAL RESULT
- Shipped change:
- Behavior improvement proven by:
- Regressions checked:
- Remaining risk:
- Why this is not just bloat:
```
