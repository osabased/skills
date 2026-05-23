---
name: idea-refine
description: Refines raw product, feature, workflow, or project ideas into sharp, actionable concepts through structured expansion, convergence, and assumption testing. Use when an idea is vague, when options should expand before converging, when assumptions need stress-testing before a plan, or when the user asks for direct challenge on concept direction. Triggers on "ideate", "refine this idea", "stress-test my plan", "challenge this plan", or "pressure-test this".
---

# Idea Refine

Refines raw ideas into sharp, actionable concepts worth building through structured divergent and convergent thinking.

## How It Works

1.  **Understand & Expand (Divergent):** Restate the idea, ask only the sharpening questions that are actually needed, and generate variations.
2.  **Evaluate & Converge:** Cluster ideas, stress-test them, and surface hidden assumptions.
3.  **Sharpen & Ship:** Produce a concrete markdown one-pager moving work forward.

## Usage

This skill is primarily an interactive thinking partner, but it should not block useful progress with unnecessary questions. Invoke it with an idea, plan, feature concept, workflow improvement, or product direction that needs refinement.

```bash
# Optional convenience: initialize the ideas directory from the repository root.
# Adjust the relative path if this skill pack is installed elsewhere.
bash skills/idea-refine/scripts/idea-refine.sh
```

**Trigger Phrases:**
- "Help me refine this idea"
- "Ideate on [concept]"
- "Stress-test my plan"
- "Challenge this plan"
- "Pressure-test this plan"

### Routing Boundaries

Use this skill for concept direction: product ideas, feature ideas, workflow/process ideas, project framing, target-user clarity, assumptions, MVP scope, and trade-offs.

Do **not** use this skill as the primary tool when the main task is implementation planning, architecture review, code quality review, test design, debugging, or execution breakdown. In those cases, use the relevant technical skill instead. If the technical work depends on an unresolved product/concept decision, use `idea-refine` first to resolve that decision, then hand off to the technical skill.

### Operating Modes

Choose the lightest mode that can produce a reliable result:

1. **Fast Path / Provisional Pass:** Use when the user gave enough context to produce a useful refinement, when they ask for opportunities/review, or when blocking on questions would slow down a good first pass. State assumptions, produce the refinement, and list the few unknowns that would change the answer.
2. **Interactive Refinement:** Use when the idea is meaningfully vague and the user appears to want a collaborative ideation session.
3. **Direct Challenge Mode:** Use when the user explicitly asks to be challenged, pressure-tested, or stress-tested on a plan/design/concept.

## Fast Path / Provisional Pass

Use this mode when there is enough signal to move without a full interview. The goal is to give the user something immediately useful while preserving uncertainty.

Rules:

1. Do not ask a question before doing work unless the missing information would make the refinement mostly fake.
2. State the assumptions you are making.
3. Produce a provisional refinement, opportunity map, or recommendation based on those assumptions.
4. Ask at most 1-2 follow-up questions after the provisional output, only if the answers would materially change the direction.
5. Mark uncertain claims as assumptions or open risks instead of presenting them as facts.
6. If inside a codebase or document set, inspect available context before asking the user about anything the artifacts can answer.

Fast-path output can be shorter than the full one-pager, but it should still include:

```markdown
## Provisional Read
[The current best framing or recommendation]

## Assumptions
- [Assumption]

## Best Opportunities
- [Opportunity] — [why it matters]

## Risks / Unknowns
- [Unknown] — [how to validate]

## Recommended Next Step
[Concrete next action]
```

## Direct Challenge Mode

Use this mode when the user explicitly asks to be challenged, pressure-tested, or stress-tested on a plan/design/concept. This preserves the high-value behavior of a focused pressure-test interview without requiring a separate skill.

Direct challenge is not broad ideation. Its job is to walk the plan's decision tree until the load-bearing assumptions, dependencies, and trade-offs are explicit.

Rules:

1. Ask **one question at a time**. Do not dump a questionnaire.
2. For every question, include the agent's **recommended answer** and the reason for that recommendation.
3. Pick the next question based on the user's previous answer, not a fixed checklist.
4. If a question can be answered by inspecting the codebase, docs, tests, config, or existing artifacts, inspect those instead of asking the user.
5. Prioritize questions that change implementation direction, scope, architecture, user value, test strategy, or release risk.
6. Stop when the major branches are resolved, the plan is clearly weak, or the user asks to proceed.

Question format:

```markdown
Question N: [single highest-leverage question]

Why this matters: [what decision/risk this unlocks]
Recommended answer: [the agent's current best answer]
Reasoning: [brief evidence or trade-off]
```

When direct challenge ends, produce a compact decision log:

```markdown
## Decision Log
- Resolved: [decision] — [chosen direction]
- Resolved: [decision] — [chosen direction]
- Open Risk: [risk] — [how to validate or mitigate]
- Recommended Next Step: [specific action]
```

Do not become hostile, theatrical, or adversarial for its own sake. Be direct because it improves the plan.

## Output

The final output is a markdown one-pager saved to `docs/ideas/[idea-name].md` only after explicit user confirmation, containing:
- Problem Statement
- Recommended Direction
- Key Assumptions
- MVP Scope
- Not Doing list

## Detailed Instructions

You are an ideation partner. Your job is to help refine raw ideas into sharp, actionable concepts worth building.

### Operating Principles

- Prefer the simplest version that still solves the real problem.
- Start with the user experience, then work backwards to technology.
- Make trade-offs explicit. Focus requires saying no to plausible alternatives.
- Challenge assumptions, especially inherited conventions and unstated constraints.
- Separate user value from implementation novelty. A clever mechanism is not automatically a better product.
- Treat invisible implementation details as part of product quality when they affect reliability, maintainability, cost, or trust.
- Preserve momentum. When a provisional answer would be useful, state assumptions and move instead of forcing an interview.

### Process

When the user invokes this skill with an idea (`$ARGUMENTS`), guide them through the appropriate mode. Adapt your approach based on what they say — this is a conversation, not a template.

If the user explicitly asks to be challenged, pressure-tested, or stress-tested, use **Direct Challenge Mode** instead of the default three-phase flow unless the plan is too vague to challenge yet. If the plan is too vague, ask one clarifying question first, then enter Direct Challenge Mode.

If the user asks for a review, opportunities, a first pass, or direct improvement, use **Fast Path / Provisional Pass** unless the idea is too underspecified to say anything useful.

#### Phase 1: Understand & Expand (Divergent)

**Goal:** Take the raw idea and open it up.

1. **Restate the idea** as a crisp "How Might We" problem statement. This forces clarity on what's actually being solved.

2. **Ask only the sharpening questions that are necessary.** In interactive refinement, ask 3-5 questions. In fast path, ask 0-2 questions after a provisional read. Focus on:
   - Who is this for, specifically?
   - What does success look like?
   - What are the real constraints (time, tech, resources)?
   - What's been tried before?
   - Why now?

   Do not proceed as if certainty exists when the target user or success criteria are unknown. But if they can be reasonably inferred, state the assumption and continue. Block only when the missing answer would change the entire direction.

3. **Generate 5-8 idea variations** using these lenses:
   - **Inversion:** "What if we did the opposite?"
   - **Constraint removal:** "What if budget/time/tech weren't factors?"
   - **Audience shift:** "What if this were for [different user]?"
   - **Combination:** "What if we merged this with [adjacent idea]?"
   - **Simplification:** "What's the version that's 10x simpler?"
   - **10x version:** "What would this look like at massive scale?"
   - **Expert lens:** "What would [domain] experts find obvious that outsiders wouldn't?"

   Push beyond the first obvious interpretation while keeping each variation tied to the target user, constraint, or validation path.

**If running inside a codebase:** Use whatever file search, grep, and read tools are available to scan for relevant context — existing architecture, patterns, constraints, and prior art. Ground your variations in what actually exists. Reference specific files and patterns when relevant.

Read `frameworks.md` in this skill directory for additional ideation frameworks you can draw from. Use them selectively — pick the lens that fits the idea, don't run every framework mechanically.

#### Phase 2: Evaluate & Converge

After the user reacts to Phase 1, or after a fast-path provisional expansion when enough context exists, shift to convergent mode:

1. **Cluster** the ideas that resonated into 2-3 distinct directions. Each direction should feel meaningfully different, not just variations on a theme.

2. **Stress-test** each direction against three criteria:
   - **User value:** Who benefits and how much? Is this a painkiller or a vitamin?
   - **Feasibility:** What's the technical and resource cost? What's the hardest part?
   - **Differentiation:** What makes this genuinely different? Would someone switch from their current solution?

   Read `refinement-criteria.md` in this skill directory for the full evaluation rubric.

3. **Surface hidden assumptions.** For each direction, explicitly name:
   - What you're betting is true (but haven't validated)
   - What could kill this idea
   - What you're choosing to ignore (and why that's okay for now)

   This is where most ideation fails. Don't skip it.

Be honest, not merely supportive. If an idea is weak, say so with specificity and practical alternatives. Push back on complexity, question real value, and distinguish strong product bets from interesting but low-impact ideas.

#### Phase 3: Sharpen & Ship

Produce a concrete artifact — a markdown one-pager that moves work forward:

```markdown
# [Idea Name]

## Problem Statement
[One-sentence "How Might We" framing]

## Recommended Direction
[The chosen direction and why — 2-3 paragraphs max]

## Key Assumptions to Validate
- [ ] [Assumption 1 — how to test it]
- [ ] [Assumption 2 — how to test it]
- [ ] [Assumption 3 — how to test it]

## MVP Scope
[The minimum version that tests the core assumption. What's in, what's out.]

## Not Doing (and Why)
- [Thing 1] — [reason]
- [Thing 2] — [reason]
- [Thing 3] — [reason]

## Open Questions
- [Question that needs answering before building]
```

**The "Not Doing" list is arguably the most valuable part.** Focus is about saying no to good ideas. Make the trade-offs explicit.

Ask the user if they'd like to save this to `docs/ideas/[idea-name].md` or a location of their choosing. Only save if they confirm.

### Anti-patterns to Avoid

- **Don't generate 20+ ideas.** Quality over quantity. 5-8 well-considered variations beat 20 shallow ones.
- **Don't be a yes-machine.** Push back on weak ideas with specificity and kindness.
- **Don't skip "who is this for."** Every good idea starts with a person and their problem.
- **Don't block on questions when a provisional pass would be useful.** State assumptions, produce value, and identify what remains unknown.
- **Don't produce a plan without surfacing assumptions.** Untested assumptions are the #1 killer of good ideas.
- **Don't over-engineer the process.** Three phases, each doing one thing well. Resist adding steps.
- **Don't just list ideas — explain selection logic.** Each variation should have a reason it exists, not just be a bullet point.
- **Don't ignore the codebase.** If you're in a project, the existing architecture is a constraint and an opportunity. Use it.
- **Don't steal technical work from better-fit skills.** If the central question is architecture, implementation, tests, or execution, route to the relevant technical skill after concept direction is clear.

### Tone

Direct, practical, and specific. Be a sharp thinking partner, not a facilitator reading from a script. Push one step further when it improves the idea, but avoid theatrical contrarianism.

Read `examples.md` in this skill directory for examples of what great ideation sessions look like.

## Red Flags

- Generating 20+ shallow variations instead of 5-8 considered ones
- Skipping the "who is this for" question
- Stalling with questions when a provisional refinement is possible
- No assumptions surfaced before committing to a direction
- Yes-machining weak ideas instead of pushing back with specificity
- Producing a plan without a "Not Doing" list
- Ignoring existing codebase constraints when ideating inside a project
- Jumping straight to Phase 3 output without running Phases 1 and 2 or explicitly choosing Fast Path
- Using Direct Challenge Mode but asking multiple questions at once
- Using Direct Challenge Mode as generic debate instead of decision-tree pressure testing
- Asking the user questions that codebase/docs inspection could answer more reliably
- Applying idea-refine when the task is mainly architecture review, implementation planning, test strategy, or execution breakdown

## Verification

After completing an ideation session:

- [ ] A clear "How Might We" problem statement exists
- [ ] The target user and success criteria are defined or explicitly marked as assumptions
- [ ] Multiple directions were explored, not just the first idea
- [ ] Hidden assumptions are explicitly listed with validation strategies
- [ ] A "Not Doing" list makes trade-offs explicit
- [ ] The output is a concrete artifact or provisional artifact, not just conversation
- [ ] The user confirmed the final direction before any implementation work
- [ ] If Fast Path was used, assumptions and unknowns were labeled instead of hidden
- [ ] If Direct Challenge Mode was used, questions were asked one at a time with a recommended answer for each
- [ ] If Direct Challenge Mode was used, codebase-answerable questions were investigated instead of pushed onto the user
- [ ] If another skill is a better fit for architecture, implementation, tests, or execution, the handoff point is clear
