---
name: write-a-skill
description: Creates, audits, revises, or packages reusable agent skills while preserving routing behavior and using the smallest fitting structure. Use when the user asks to create, edit, normalize, audit, review, or package a skill, especially when triggers, workflow steps, supporting files, verification gates, or template fit need to be decided.
---

# Writing Skills

## Overview

Create skills as executable operating guidance, not reference essays. A good skill gives an agent reliable routing, bounded use, concrete execution steps, misuse guardrails, and evidence requirements.

The standard workflow anatomy is the default shape for reusable process skills. It is not a universal mold. Router skills, meta-skills, diagnostic checklists, compact utilities, and reference-heavy helpers may need a smaller structure. Do not force a fake workflow onto a skill when that would add ceremony, change routing behavior, or obscure the skill's actual job.

`SKILL.md` is the entry point. Keep it focused on the common operating path. Move deep reference material, long examples, commands, schemas, extended checklists, and edge cases into supporting files that are loaded only when needed.

## When to Use

- User wants to create a new reusable skill
- User wants to revise, normalize, audit, or package an existing skill
- A skill lacks clear trigger conditions, safe-use boundaries, or evidence gates
- A workflow skill is missing concrete steps, rationalizations, red flags, or verification
- A skill reads like documentation instead of agent-followable operating guidance
- A skill is too large and needs progressive disclosure through supporting files
- A skills pack change might affect routing, activation, or cross-skill behavior

**When NOT to use:** Do not create a skill for one-off advice, project-specific notes, or information that will become stale quickly. Do not mass-normalize unrelated skills just because a format exists. Use a skill only when the guidance is reusable across future tasks.

## The Skill Creation Workflow

```
1. CLASSIFY  → Identify the reusable task, trigger conditions, exclusions, and proof of value
2. SCOPE     → Decide what belongs in SKILL.md vs supporting files
3. IMPACT    → Check routing, workflow weight, boundaries, and pack-level blast radius
4. FIT       → Choose the smallest structure that fits the skill shape
5. DRAFT     → Write or revise the skill without replacing its domain logic
6. PACKAGE   → Place files, links, and optional scripts in a valid folder structure
7. VERIFY    → Confirm triggers, links, behavior preservation, evidence requirements, and behavior eval when the change affects skill execution
```

### Step 1: Classify the Skill

Before writing, answer:

- What reusable task does this skill guide?
- What user requests should trigger it?
- What should explicitly not trigger it?
- Is this a workflow agents perform, a router that selects other skills, a diagnostic checklist, a utility, or reference guidance?
- What artifact, decision, command, test, or evidence proves the skill did useful work?

If the task is not reusable, do not create a skill. Give the user a direct answer or create project documentation instead.

### Step 2: Scope the Skill

Keep `SKILL.md` as the operational path. Add supporting files only when they reduce noise in the entry point or prevent the common path from becoming reference-heavy.

Before changing an existing skill, preserve its original job unless the user explicitly asked to change that job. Format changes should improve activation, execution, and verification without silently changing the skill's scope, strictness, or relationship to other skills.

Use this package structure as a menu of allowed parts, not a checklist. Start with only `SKILL.md`, then add supporting files only when they earn their keep:

```
skill-name/
├── SKILL.md              # Required entry point
├── TEMPLATE.md           # Copyable skeletons, if the skill teaches repeatable formats
├── EVALUATION.md         # Gold-task behavior checks for meaningful skill changes
├── REFERENCE.md          # Deep reference material, if needed
├── EXAMPLES.md           # Longer examples, if needed
├── CHECKLISTS.md         # Extended checklists, if needed
└── scripts/              # Deterministic helper scripts, if needed
    └── helper.sh
```

Split supporting files when:

- `SKILL.md` is becoming reference-heavy instead of workflow-heavy
- content is advanced, rare, or optional
- examples, commands, schemas, or edge cases are long
- the skill covers distinct subdomains that should not always load
- a template or checklist is useful to copy but would distract from the operating path

### Step 3: Check System Impact

Before applying a template to an existing skill or skills pack, check downstream effects. A format improvement is bad if it changes routing behavior, makes every task heavier, or teaches agents to apply ceremony instead of judgment.

| Check | Question | Bad Outcome to Avoid |
|---|---|---|
| Trigger scope | Does the description activate the skill only for the right requests? | Skill fires for broad, unrelated tasks |
| Workflow weight | Did the edit add required steps that make simple tasks slower? | Agents perform ceremony instead of work |
| Skill boundaries | Did the edit steal responsibility from another skill? | Router ambiguity and duplicated workflows |
| Progressive disclosure | Is common-path guidance still in `SKILL.md` while deep material is optional? | Entry file becomes a reference dump |
| Verification burden | Are evidence requirements concrete and proportional? | Impossible, vague, or performative verification gates |
| Existing behavior | Did wording change the skill's actual policy, not just its structure? | Accidental new constraints, permissions, or exclusions |

When the answer is uncertain, make the smaller edit. Prefer preserving behavior and adding one clear guardrail over rewriting a skill around a new theory.

### Step 4: Choose the Smallest Fitting Structure

Classify the skill shape before applying the standard workflow anatomy. The template should serve the skill's job, not replace it.

| Skill Shape | Best Structure | Do Not Force |
|---|---|---|
| Workflow skill | Overview, When to Use, ordered workflow, rationalizations, red flags, verification | Extra sections that do not change execution |
| Router or meta-skill | Purpose, routing rules, selection criteria, boundaries, minimal verification | A fake task workflow for every routed skill |
| Reference-heavy helper | Scope, when to consult, compact usage rules, reference map, evidence that the reference affected a decision | Step-by-step ceremony when the task is lookup or judgment support |
| Utility or script skill | Inputs, invocation rules, safety limits, outputs, failure handling, verification | Long conceptual background |
| Diagnostic checklist | Symptoms, checks, evidence thresholds, escalation rules, verification | A strict linear workflow when check order is not required |

Use the full workflow anatomy only when it makes activation, execution, or verification more reliable. Otherwise, preserve the smallest structure that still makes the skill safe and useful.

### Step 5: Draft the Skill

Use the workflow anatomy as the default for reusable workflow skills. Apply it proportionally: preserve domain logic, avoid filler, and do not let a template override a better minimal structure.

For copyable workflow and non-workflow skeletons, see [`TEMPLATE.md`](TEMPLATE.md). Use the reference file after deciding the skill shape, not as a reason to make every skill look the same.

#### Format Rules

- Use frontmatter with `name` and `description`.
- Write `description` as the routing hint. It must say what the skill does and when to use it.
- Put the core operating path in `SKILL.md`, not only in supporting files.
- Use active step names: `Measure`, `Classify`, `Map`, `Verify`, `Guard`.
- Include a clear `When NOT to use` line when misuse or overuse is plausible.
- Include `Common Rationalizations`, `Red Flags`, and `Verification` in workflow skills. For non-workflow skills, include only the misuse guardrails and evidence checks that materially improve routing, safe use, or output quality.
- Prefer compact tables, checklists, and decision trees over long prose.
- Link supporting files from `See Also`; do not force-load deep references in the main file.

#### Description Requirements

The description is the routing surface. Agents may see only the description before deciding whether to load the skill.

Requirements:

- Maximum 1024 characters
- Third person
- First sentence says what the skill does
- Includes `Use when...` with concrete trigger conditions
- Mentions file types, task types, or risk signals when relevant
- Narrows activation instead of inflating the skill's importance

Good:

```
description: Guides agents through event-driven architecture decisions, event contracts, topology selection, and reliability design. Use when designing async events, pub/sub, queues, streams, sagas, outbox/inbox, projections, or event replay.
```

Bad:

```
description: Helps with architecture.
```

### Step 6: Package with Progressive Disclosure

Use supporting files only when they improve readability, reuse, or reliability.

| File Type | Use For |
|---|---|
| `TEMPLATE.md` | Copyable skeletons for workflow skills and compact non-workflow skill shapes |
| `EVALUATION.md` | Lightweight gold-task behavior checks for new or behavior-changing skill edits |
| `REFERENCE.md` | Detailed concepts, background, or taxonomy |
| `EXAMPLES.md` | Longer worked examples or before/after examples |
| `CHECKLISTS.md` | Extended audit or review lists |
| `COMMANDS.md` | Tool-specific commands and invocation examples |
| `SCHEMAS.md` | JSON/YAML/API schemas or contract formats |
| `scripts/` | Deterministic helpers that reduce repeated generated code |

Supporting files should be referenced from `SKILL.md` but should not be required for the normal path.

Add scripts only when they improve reliability or reduce repeated code generation.

Good script candidates:

- validation commands
- formatters
- file structure checks
- deterministic transformations
- repeatable report generation

Do not add scripts for subjective work, architectural judgment, or tasks that need human/contextual reasoning.

### Step 7: Verify Behavior and Structure

Run the verification checklist before returning a skill or package. For existing skills, explicitly check that the edit preserved intended routing and did not introduce pack-wide policy changes.

For behavior-changing skill work, add a small gold-task evaluation using [`EVALUATION.md`](EVALUATION.md). This is required when creating a new skill, changing trigger conditions, changing workflow steps, adding/removing support files that affect execution, changing a router/meta-skill, or claiming a quality/reliability improvement.

Do not run the eval module for typo fixes, broken-link repairs, formatting-only edits, or changes that cannot affect when the skill fires or what the agent does.

A valid behavior eval shows:

- an obvious trigger case,
- an edge trigger case,
- a should-not-trigger case,
- a cross-skill routing case when pack interaction is affected,
- a regression case for behavior that must stay intact.

If the eval does not show a concrete behavior improvement, prefer a smaller patch or no change.

If the skill is packaged as an archive, inspect the archive contents after creating it. A valid package must contain the expected folder path, `SKILL.md`, referenced support files, and no accidental workspace or cache files.

## See Also

For copyable skeletons for workflow, router, reference-heavy, utility, and diagnostic skill shapes, see [`TEMPLATE.md`](TEMPLATE.md).

For lightweight gold-task checks that prove skill behavior changed in a useful way, see [`EVALUATION.md`](EVALUATION.md).

## Change Safety Rules

When revising skills, optimize for correctness and routing stability before consistency. Consistency is valuable only when it makes agents choose and execute the right workflow more reliably.

Rules:

- Do not rewrite every skill in a pack unless the user asked for a pack-wide normalization.
- Do not add a supporting file unless it removes meaningful complexity from `SKILL.md` or improves reuse.
- Do not make optional best practices sound mandatory unless skipping them would make the skill unsafe or useless.
- Do not broaden the description to make the skill sound more important.
- Do not add cross-skill handoffs unless the other skill materially changes the result.
- Do not replace a skill's domain-specific workflow with a generic template skeleton.
- Do not make non-workflow skills carry workflow-only sections unless those sections change behavior or evidence quality.

A good format pass should leave the skill easier to trigger, easier to follow, and harder to fake — not bigger by default.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The skill just needs information; a workflow is unnecessary." | Skills are for agent behavior. If the content only stores facts, it may belong in reference docs. If it changes agent decisions or execution, choose the smallest structure that supports that behavior. |
| "The description can be vague because the title is clear." | The description is the routing hint. Vague descriptions cause missed or incorrect skill activation. |
| "I'll skip rationalizations; they're obvious." | Agents repeatedly skip obvious discipline. Naming excuses makes the skill harder to bypass. |
| "Verification depends on the task, so I'll leave it generic." | Verification can be domain-specific. Require the artifact, command, test, decision record, or evidence that proves completion. |
| "Everything should go in SKILL.md so the agent sees it." | Overloaded entry files waste context. Keep the common path in `SKILL.md` and link deeper references only when needed. |
| "This one-off project note should be a skill." | Skills should be reusable. Project-specific notes belong in project docs, plans, or handoffs. |
| "I'll normalize the whole pack while I'm here." | Pack-wide edits can change routing and behavior. Change only the requested skill unless a deliberate pack migration is requested. |
| "The template says every section is required, so I'll fill it even if it adds no value." | The anatomy is a default operating shape for workflow skills, not permission for filler. Each section must change agent behavior or verification quality. |
| "The workflow template should apply to every skill for consistency." | Consistency is useful only when it improves routing and execution. Router, reference, diagnostic, and utility skills may need different structures. |

## Red Flags

- `SKILL.md` reads like an article instead of an operating procedure
- no clear trigger conditions in the description
- no `When NOT to use` boundary when overuse is plausible
- workflow skill has no anti-rationalization table, red flags, or verification gate
- non-workflow skill is padded with fake workflow sections
- verification says only "reviewed" or "seems correct"
- supporting files are required for the normal path
- every edge case is included in the entry file
- skill creates abstractions without a repeatable use case
- format pass changes when a skill activates without acknowledging the behavior change
- supporting files multiply without a clear token, reuse, or reliability benefit
- template compliance replaces the skill's domain-specific decision logic

## Verification

After creating or revising a skill:

- [ ] `SKILL.md` has valid frontmatter with `name` and `description`
- [ ] `description` includes specific `Use when...` triggers and stays under 1024 characters
- [ ] the overview or equivalent scope section states the core operating principle
- [ ] positive triggers and misuse boundaries are present where needed
- [ ] workflow steps are concrete and ordered for workflow skills
- [ ] non-workflow skills use a structure that matches their job
- [ ] rationalizations, red flags, or equivalent misuse guardrails exist where they improve behavior
- [ ] verification requirements fit the skill shape and require evidence, not vibes
- [ ] supporting files are linked from `See Also` or the relevant usage section
- [ ] package structure is valid and real, non-placeholder references resolve
- [ ] existing skill scope and routing behavior were preserved or intentionally changed
- [ ] behavior-changing edits used a lightweight gold-task eval or explicitly did not need one
- [ ] no pack-wide convention was introduced accidentally
- [ ] template fit was assessed before applying the workflow anatomy
- [ ] no new section is filler; each section affects behavior, judgment, routing, or evidence
