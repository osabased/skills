---
name: source-driven-development
description: Grounds version-sensitive, unfamiliar, reusable, or production-critical framework/library decisions in official documentation. Use when framework/library behavior may be stale, version-specific, copied across the project, or explicitly needs source-backed correctness.
---

# Source-Driven Development

## Overview

Source-driven development backs framework and library decisions with official documentation when documentation materially changes correctness. It is not a mandate to fetch docs for every line of framework code. Use it when APIs, conventions, versions, deprecations, or reusable patterns could make memory-based implementation wrong.

Training data goes stale, APIs get deprecated, and best practices evolve. This skill keeps high-consequence or reusable code trustworthy because the pattern traces back to an authoritative source the user can check.

## When to Use

Use this skill when at least one of these is true:

- Framework or library behavior is version-sensitive, newly released, deprecated, or likely to have changed
- Building boilerplate, starter code, shared utilities, wrappers, or patterns that will be copied across a project
- The user explicitly asks for documented, verified, source-backed, or "correct" implementation
- Implementing features where the framework's recommended approach materially matters, such as forms, routing, data fetching, state management, auth, migrations, build config, or plugin setup
- Reviewing or improving framework-specific code that may rely on stale, deprecated, or non-recommended patterns
- Production-facing behavior depends on a framework/library API working exactly as assumed
- You are unfamiliar with the framework/library version or cannot confidently identify the current recommended pattern

**When NOT to use:**

- Correctness does not depend on a specific version, such as renaming variables, fixing typos, or moving files
- Pure logic that works the same across all versions, such as loops, conditionals, and ordinary data structures
- The change only follows an obvious local convention already established in nearby code
- The framework/library use is routine, low-risk, and not being turned into a reusable pattern
- The user explicitly wants speed over verification, such as "just do it quickly"

## The Process

```
DETECT ──→ FETCH ──→ IMPLEMENT ──→ CITE
  │          │           │            │
  ▼          ▼           ▼            ▼
 What       Get the    Follow the   Show your
 stack?     relevant   documented   sources
            docs       patterns
```

### Step 1: Detect Stack and Versions

When the selected decision depends on framework/library behavior, read the project's dependency file to identify exact versions:

```
package.json    → Node/React/Vue/Angular/Svelte
composer.json   → PHP/Symfony/Laravel
requirements.txt / pyproject.toml → Python/Django/Flask
go.mod          → Go
Cargo.toml      → Rust
Gemfile         → Ruby/Rails
```

State what you found explicitly:

```
STACK DETECTED:
- React 19.1.0 (from package.json)
- Vite 6.2.0
- Tailwind CSS 4.0.3
→ Fetching official docs for the relevant patterns.
```

If versions are missing or ambiguous and the decision is version-sensitive, ask the user or state the assumption before implementing. Don't pretend a guessed version is evidence.

### Step 2: Fetch Official Documentation

Fetch the specific documentation page for the feature you're implementing. Not the homepage, not the full docs — the relevant page.

**Source hierarchy (in order of authority):**

| Priority | Source | Example |
|----------|--------|---------|
| 1 | Official documentation | react.dev, docs.djangoproject.com, symfony.com/doc |
| 2 | Official blog / changelog | react.dev/blog, nextjs.org/blog |
| 3 | Web standards references | MDN, web.dev, html.spec.whatwg.org |
| 4 | Browser/runtime compatibility | caniuse.com, node.green |

**Not authoritative — never cite as primary sources:**

- Stack Overflow answers
- Blog posts or tutorials (even popular ones)
- AI-generated documentation or summaries
- Your own training data (that is the whole point — verify it)

**Be precise with what you fetch:**

```
BAD:  Fetch the React homepage
GOOD: Fetch react.dev/reference/react/useActionState

BAD:  Search "django authentication best practices"
GOOD: Fetch docs.djangoproject.com/en/6.0/topics/auth/
```

After fetching, extract the key patterns and note any deprecation warnings or migration guidance.

When official sources conflict with each other, such as a migration guide contradicting the API reference, surface the discrepancy to the user and verify which pattern actually works against the detected version.

### Step 3: Implement Following Documented Patterns

Write code that matches what the documentation shows:

- Use the API signatures from the docs, not from memory
- If the docs show a new way to do something, use the new way
- If the docs deprecate a pattern, don't use the deprecated version
- If the docs don't cover something, flag it as unverified
- If local project conventions intentionally differ from the docs, surface the trade-off instead of silently overriding the codebase

**When docs conflict with existing project code:**

```
CONFLICT DETECTED:
The existing codebase uses useState for form loading state,
but React 19 docs recommend useActionState for this pattern.
(Source: react.dev/reference/react/useActionState)

Options:
A) Use the modern pattern (useActionState) — consistent with current docs
B) Match existing code (useState) — consistent with codebase
→ Which approach do you prefer?
```

Surface the conflict. Don't silently pick one.

### Step 4: Cite Your Sources

Non-trivial framework-specific decisions get citations. The user must be able to verify decisions where documentation affected the implementation.

**In code comments, only when the citation helps future maintainers:**

```typescript
// React 19 form handling with useActionState
// Source: https://react.dev/reference/react/useActionState#usage
const [state, formAction, isPending] = useActionState(submitOrder, initialState);
```

Do not litter routine code with documentation comments. Prefer conversation/report citations unless the source explains a non-obvious, version-specific, or unusual pattern.

**In conversation:**

```
I'm using useActionState instead of manual useState for the
form submission state because React 19 documents actions as the
recommended pattern for this flow.

Source: https://react.dev/reference/react/useActionState
```

**Citation rules:**

- Full URLs, not shortened
- Prefer deep links with anchors where possible, such as `/useActionState#usage` over `/useActionState`
- Quote the relevant passage only when it supports a non-obvious decision
- Include browser/runtime support data when recommending platform features
- If you cannot find documentation for a pattern, say so explicitly:

```
UNVERIFIED: I could not find official documentation for this
pattern. This is based on training data and may be outdated.
Verify before using in production.
```

Honesty about what you couldn't verify is more valuable than false confidence.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'm confident about this API" | Confidence is not evidence when versions, deprecations, or framework recommendations matter. Verify the specific pattern. |
| "Fetching docs wastes tokens" | Fetching targeted docs is cheaper than debugging a stale API. Fetch the relevant page, not the whole docs site. |
| "The docs won't have what I need" | If the docs don't cover it, that's valuable information — the pattern may not be officially recommended. |
| "I'll just mention it might be outdated" | A disclaimer doesn't help. Either verify and cite, or clearly flag it as unverified. |
| "This is framework code, so I must fetch docs" | Not always. Routine local edits, obvious project conventions, and version-insensitive code do not need a documentation expedition. |
| "This is a simple task, no need to check" | Simple reusable patterns can become templates. Verify when the pattern will be copied, shared, or used as project precedent. |

## Red Flags

- Writing version-sensitive or reusable framework-specific code without checking the docs for that version
- Using "I believe" or "I think" about a changing API instead of citing the source
- Implementing a pattern without knowing which version it applies to
- Citing Stack Overflow or blog posts instead of official documentation
- Using deprecated APIs because they appear in training data
- Not reading dependency files before implementing a version-dependent decision
- Delivering source-backed code without source citations for the decisions docs actually affected
- Fetching an entire docs site when only one page is relevant
- Fetching docs for routine local edits where the source would not change the implementation

## Verification

After implementing with source-driven development:

- [ ] Framework and library versions were identified when the decision depended on version behavior
- [ ] Official documentation was fetched for version-sensitive, unfamiliar, reusable, or production-critical framework/library patterns
- [ ] All cited sources are official documentation, official changelogs/blogs, web standards references, or compatibility references
- [ ] Code follows the patterns shown in the relevant current documentation, unless an explicit local trade-off was documented
- [ ] Non-trivial decisions include source citations with full URLs
- [ ] Deprecated APIs are avoided or explicitly justified against project constraints
- [ ] Conflicts between docs and existing code were surfaced to the user
- [ ] Anything that could not be verified is explicitly flagged as unverified
