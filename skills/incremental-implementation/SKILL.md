---
name: incremental-implementation
description: Delivers risky or hard-to-reason-about changes incrementally. Use for multi-file work where behavior, dependencies, rollback, or validation would become difficult if implemented in one pass.
---

# Incremental Implementation

## Overview

Build in thin vertical slices — implement one piece, test it, verify it, then expand. Avoid implementing an entire feature in one pass when the change has enough behavioral scope, dependency risk, or rollback complexity that a single wide edit would be hard to reason about.

Each increment should leave the system in a working, testable state. This is the execution discipline that makes substantial changes manageable without adding ceremony to trivial edits.

## When to Use

Use this skill when at least one of these is true:

- A multi-file change includes behavior, dependencies, risk, or rollback concerns that would be hard to reason about in one pass
- Building a new feature from a task breakdown
- Refactoring existing code where correctness must be preserved across multiple call sites or modules
- The implementation crosses boundaries such as UI/API/database, client/server, package/module, or config/runtime
- You are tempted to write more than ~100 lines before testing
- A failed approach would be expensive to unwind unless progress is checkpointed

**When NOT to use:**

- Single-file, single-function changes where the scope is already minimal
- Mechanical multi-file edits such as renames, import updates, formatting, or obvious prop/type threading
- Small two-file changes where behavior is clear, rollback is trivial, and validation is immediate
- Pure documentation or comment edits

## Slice Setup

Before coding, write a tiny story card for the first slice when the change is not already obvious from the task:

```text
STORY CARD
- User-visible behavior:
- Acceptance examples:
- Risk/uncertainty:
- Out of scope:
- First slice:
```

Choose the first slice by either:

1. the smallest user-visible behavior that proves the path works, or
2. the highest uncertainty that could invalidate the approach.

Do not start with broad infrastructure unless it directly enables the first verified behavior. This setup stops implementation from becoming a wide speculative edit before the behavior, boundary, and first proof point are clear.

For tiny low-risk changes, skip the story card and just make the change with immediate verification.

## The Increment Cycle

```
┌──────────────────────────────────────┐
│                                      │
│   Implement ──→ Test ──→ Verify ──┐  │
│       ▲                           │  │
│       └───── Commit ◄─────────────┘  │
│              │                       │
│              ▼                       │
│          Next slice                  │
│                                      │
└──────────────────────────────────────┘
```

For each slice:

1. **Implement** the smallest complete piece of functionality
2. **Test** — run the relevant test suite, or write a test if none exists and the change warrants one
3. **Verify** — confirm the slice works as expected through tests, build, typecheck, lint, or manual check as appropriate
4. **Commit** -- save progress with a descriptive message using the local commit guidance below when commits are available and useful
5. **Move to the next slice** — carry forward, don't restart

## Slicing Strategies

### Vertical Slices (Preferred)

Build one complete path through the stack:

```
Slice 1: Create a task (DB + API + basic UI)
    → Tests pass, user can create a task via the UI

Slice 2: List tasks (query + API + UI)
    → Tests pass, user can see their tasks

Slice 3: Edit a task (update + API + UI)
    → Tests pass, user can modify tasks

Slice 4: Delete a task (delete + API + UI + confirmation)
    → Tests pass, full CRUD complete
```

Each slice delivers working end-to-end functionality.

### Contract-First Slicing

When backend and frontend need to develop in parallel:

```
Slice 0: Define the API contract (types, interfaces, OpenAPI spec)
Slice 1a: Implement backend against the contract + API tests
Slice 1b: Implement frontend against mock data matching the contract
Slice 2: Integrate and test end-to-end
```

### Risk-First Slicing

Tackle the riskiest or most uncertain piece first:

```
Slice 1: Prove the WebSocket connection works (highest risk)
Slice 2: Build real-time task updates on the proven connection
Slice 3: Add offline support and reconnection
```

If Slice 1 fails, you discover it before investing in Slices 2 and 3.

## Implementation Rules

### Rule 0: Simplicity First

Before writing any code, ask: "What is the simplest thing that could work?"

After writing code, review it against these checks:
- Can this be done in fewer lines?
- Are these abstractions earning their complexity?
- Would a staff engineer look at this and say "why didn't you just..."?
- Am I building for hypothetical future requirements, or the current task?

```
SIMPLICITY CHECK:
✗ Generic EventBus with middleware pipeline for one notification
✓ Simple function call

✗ Abstract factory pattern for two similar components
✓ Two straightforward components with shared utilities

✗ Config-driven form builder for three forms
✓ Three form components
```

Three similar lines of code is better than a premature abstraction. Implement the naive, obviously-correct version first. Optimize only after correctness is proven with tests.

If the simple implementation creates obvious duplication, repeated maintenance, awkward seams, or broad ripple effects, do not silently overbuild. Keep the current slice small, verify it, then route to `improve-codebase-architecture` if the design problem is real. This preserves incremental delivery while still catching the point where local simplicity has become architectural debt.

### Rule 0.5: Scope Discipline

Touch only what the task requires.

Do NOT:
- "Clean up" code adjacent to your change
- Refactor imports in files you're not modifying
- Remove comments you don't fully understand
- Add features not in the spec because they "seem useful"
- Modernize syntax in files you're only reading

If you notice something worth improving outside your task scope, note it — don't fix it:

```
NOTICED BUT NOT TOUCHING:
- src/utils/format.ts has an unused import (unrelated to this task)
- The auth middleware could use better error messages (separate task)
→ Want me to create tasks for these?
```

### Rule 0.75: Local Commit Guidance

Use local guidance rather than relying on another skill pack:

- Commit only after the slice is verified.
- Keep one logical change per commit.
- Use a message that names the behavior, fix, or refactor completed.
- Do not mix feature work, cleanup, config changes, and unrelated refactors in one commit.
- If commits are unavailable in the environment, produce a commit-ready summary with changed files, verification run, and the exact logical change completed.

This keeps the skill self-contained instead of depending on commit guidance from another skill pack.

### Rule 1: One Thing at a Time

Each increment changes one logical thing. Don't mix concerns:

**Bad:** One commit that adds a new component, refactors an existing one, and updates the build config.

**Good:** Three separate commits — one for each change.

For low-risk mechanical edits, one verified commit is fine. Do not split work merely to satisfy the ritual of slicing.

### Rule 2: Keep It Compilable

After each meaningful increment, the project must build and existing tests must pass. Don't leave the codebase in a broken state between slices.

### Rule 3: Feature Flags for Incomplete Features

If a feature isn't ready for users but you need to merge increments:

```typescript
// Feature flag for work-in-progress
const ENABLE_TASK_SHARING = process.env.FEATURE_TASK_SHARING === 'true';

if (ENABLE_TASK_SHARING) {
  // New sharing UI
}
```

This lets you merge small increments to the main branch without exposing incomplete work.

### Rule 4: Safe Defaults

New code should default to safe, conservative behavior:

```typescript
// Safe: disabled by default, opt-in
export function createTask(data: TaskInput, options?: { notify?: boolean }) {
  const shouldNotify = options?.notify ?? false;
  // ...
}
```

### Rule 5: Rollback-Friendly

Each substantial increment should be independently revertable:

- Additive changes (new files, new functions) are easy to revert
- Modifications to existing code should be minimal and focused
- Database migrations should have corresponding rollback migrations
- Avoid deleting something in one commit and replacing it in the same commit — separate them when the risk warrants it

## Working with Agents

When directing an agent to implement incrementally:

```
"Let's implement Task 3 from the plan.

Start with just the database schema change and the API endpoint.
Don't touch the UI yet — we'll do that in the next increment.

After implementing, run `npm test` and `npm run build` to verify
nothing is broken."
```

Be explicit about what's in scope and what's NOT in scope for each increment.

## Increment Checklist

After each meaningful increment, verify:

- [ ] The change does one thing and does it completely
- [ ] Relevant existing tests still pass
- [ ] Relevant build/typecheck/lint checks succeed when the change could affect them
- [ ] The new functionality works as expected
- [ ] The change is committed or summarized as a commit-ready logical unit when commits are unavailable

**Note:** Run each verification command after a change that could affect it. After a successful run, don't repeat the same command unless the code has changed since — re-running on unchanged code adds no information.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll test it all at the end" | Bugs compound. A bug in Slice 1 makes Slices 2-5 wrong. Test meaningful slices. |
| "It's faster to do it all at once" | It *feels* faster until something breaks and you can't find which of 500 changed lines caused it. |
| "This touches two files, so I need a full slicing plan" | Not necessarily. Mechanical or trivial two-file changes do not need story cards and staged delivery. |
| "These changes are too small to commit separately" | Small commits are useful when they isolate behavior or risk. Ritual tiny commits add noise. |
| "I'll add the feature flag later" | If the feature isn't complete and could be user-visible, it shouldn't be exposed. Add the flag now. |
| "This refactor is small enough to include" | Refactors mixed with features make both harder to review and debug. Separate them when they are not required by the slice. |
| "Let me run the build command again just to be sure" | After a successful run, repeating the same command adds nothing unless the code has changed since. Run it again after subsequent edits, not as reassurance. |

## Red Flags

- More than 100 lines of behavior-changing code written without running tests
- Multiple unrelated changes in a single increment
- "Let me just quickly add this too" scope expansion
- Skipping the test/verify step to move faster
- Build or tests broken between increments
- Large uncommitted changes accumulating
- Building abstractions before the third use case demands it
- Touching files outside the task scope "while I'm here"
- Creating new utility files for one-time operations
- Running the same build/test command twice in a row without any intervening code change
- Producing story cards, slice plans, or commit rituals for trivial mechanical edits

## Verification

After completing all increments for a task:

- [ ] Each meaningful increment was individually tested and committed or summarized
- [ ] The relevant test suite passes
- [ ] The build/typecheck/lint checks are clean when applicable
- [ ] The feature works end-to-end as specified
- [ ] No unrelated changes remain
