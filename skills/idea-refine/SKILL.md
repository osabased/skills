---
name: idea-refine
description: Refines raw ideas into sharp, actionable concepts through structured divergent and convergent thinking or direct challenge. Use when an idea is vague, when assumptions need stress-testing before a plan, when options should expand before converging, or when the user asks for direct challenge on a plan. Triggers on "ideate", "refine this idea", "stress-test my plan", "challenge this plan", or "pressure-test this".
---

# Idea Refine

Refines raw ideas into sharp, actionable concepts worth building through structured divergent and convergent thinking.

## How It Works

1.  **Understand & Expand (Divergent):** Restate the idea, ask sharpening questions, and generate variations.
2.  **Evaluate & Converge:** Cluster ideas, stress-test them, and surface hidden assumptions.
3.  **Sharpen & Ship:** Produce a concrete markdown one-pager moving work forward.

## Usage

This skill is primarily an interactive dialogue. Invoke it with an idea, and the agent will guide you through the process.

```bash
# Optional: initialize the ideas directory from the repository root.
# Adjust the relative path if this skill pack is installed elsewhere.
bash skills/idea-refine/scripts/idea-refine.sh
```

**Trigger Phrases:**
- "Help me refine this idea"
- "Ideate on [concept]"
- "Stress-test my plan"
- "Challenge this plan"
- "Pressure-test this plan"

## Direct Challenge Mode

Use this mode when the user explicitly asks to be challenged, pressure-tested, or stress-tested on a plan/design. This preserves the high-value behavior of a focused pressure-test interview without requiring a separate skill.

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

The final output is a markdown one-pager saved to `docs/ideas/[idea-name].md` (after user confirmation), containing:
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

### Process

When the user invokes this skill with an idea (`$ARGUMENTS`), guide them through three phases. Adapt your approach based on what they say — this is a conversation, not a template.

If the user explicitly asks to be challenged, pressure-tested, or stress-tested, use **Direct Challenge Mode** instead of the default three-phase flow unless the plan is too vague to challenge yet. If the plan is too vague, ask one clarifying question first, then enter Direct Challenge Mode.

#### Phase 1: Understand & Expand (Divergent)

**Goal:** Take the raw idea and open it up.

1. **Restate the idea** as a crisp "How Might We" problem statement. This forces clarity on what's actually being solved.

2. **Ask 3-5 sharpening questions** — no more. Focus on:
   - Who is this for, specifically?
   - What does success look like?
   - What are the real constraints (time, tech, resources)?
   - What's been tried before?
   - Why now?

   Ask the user directly to gather this input. Do NOT proceed until you understand who this is for and what success looks like.

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

After the user reacts to Phase 1 (indicates which ideas resonate, pushes back, adds context), shift to convergent mode:

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

Ask the user if they'd like to save this to `docs/ideas/[idea-name].md` (or a location of their choosing). Only save if they confirm.

### Anti-patterns to Avoid

- **Don't generate 20+ ideas.** Quality over quantity. 5-8 well-considered variations beat 20 shallow ones.
- **Don't be a yes-machine.** Push back on weak ideas with specificity and kindness.
- **Don't skip "who is this for."** Every good idea starts with a person and their problem.
- **Don't produce a plan without surfacing assumptions.** Untested assumptions are the #1 killer of good ideas.
- **Don't over-engineer the process.** Three phases, each doing one thing well. Resist adding steps.
- **Don't just list ideas — explain selection logic.** Each variation should have a reason it exists, not just be a bullet point.
- **Don't ignore the codebase.** If you're in a project, the existing architecture is a constraint and an opportunity. Use it.

### Tone

Direct, practical, and specific. Be a sharp thinking partner, not a facilitator reading from a script. Push one step further when it improves the idea, but avoid theatrical contrarianism.

Read `examples.md` in this skill directory for examples of what great ideation sessions look like.

## Red Flags

- Generating 20+ shallow variations instead of 5-8 considered ones
- Skipping the "who is this for" question
- No assumptions surfaced before committing to a direction
- Yes-machining weak ideas instead of pushing back with specificity
- Producing a plan without a "Not Doing" list
- Ignoring existing codebase constraints when ideating inside a project
- Jumping straight to Phase 3 output without running Phases 1 and 2
- Using Direct Challenge Mode but asking multiple questions at once
- Asking the user questions that codebase/docs inspection could answer more reliably

## Verification

After completing an ideation session:

- [ ] A clear "How Might We" problem statement exists
- [ ] The target user and success criteria are defined
- [ ] Multiple directions were explored, not just the first idea
- [ ] Hidden assumptions are explicitly listed with validation strategies
- [ ] A "Not Doing" list makes trade-offs explicit
- [ ] The output is a concrete artifact (markdown one-pager), not just conversation
- [ ] The user confirmed the final direction before any implementation work
- [ ] If Direct Challenge Mode was used, questions were asked one at a time with a recommended answer for each
- [ ] If Direct Challenge Mode was used, codebase-answerable questions were investigated instead of pushed onto the user
