# HTML Report Format

The architectural review is rendered as a single self-contained HTML file in the OS temp directory. The file should not require network access: use embedded CSS and inline SVG/HTML diagrams by default. Mermaid is optional only when the local environment already supports it or the user explicitly accepts CDN use. Hand-built divs and inline SVG handle the more editorial visuals well: mass diagrams, cross-sections, seam diagrams, and collapse diagrams.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <style>
      :root { color-scheme: light; }
      body { margin: 0; background: #fafaf9; color: #0f172a; font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; }
      main { max-width: 72rem; margin: 0 auto; padding: 3rem 1.5rem; }
      .card { border: 1px solid #e2e8f0; border-radius: 1rem; background: white; padding: 1rem; box-shadow: 0 1px 2px rgb(15 23 42 / 0.06); }
      .grid { display: grid; gap: 1rem; grid-template-columns: repeat(auto-fit, minmax(18rem, 1fr)); }
      .badge { display: inline-flex; align-items: center; border-radius: 999px; padding: .25rem .6rem; font-size: .75rem; font-weight: 700; text-transform: uppercase; letter-spacing: .05em; }
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; stroke-width: 2; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); color: white; }
    </style>
  </head>
  <body>
    <main>
      <header>...</header>
      <section id="candidates">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, and a compact legend: solid box = module, dashed line = seam, red arrow = leakage, thick dark box = deep module. No introduction paragraph — straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the glossary terms ([LANGUAGE.md](LANGUAGE.md)) without ceremony.

Each candidate is one `<article>`:

- **Title** — short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Badge row** — recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the dependency category (`in-process`, `local-substitutable`, `ports & adapters`, `mock`).
- **Files** — monospaced list, `font-mono text-sm`.
- **Before / After diagram** — the centrepiece. Two columns, side by side. See patterns below.
- **Problem** — one sentence. What hurts.
- **Solution** — one sentence. What changes.
- **Wins** — bullets, ≤6 words each. e.g. "Tests hit one interface", "Pricing logic stops leaking", "Delete 4 shallow wrappers".
- **ADR callout** (if applicable) — one line in an amber-tinted box.

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point.

### Graph-shaped dependencies / call flow

Use inline SVG or plain HTML boxes when the point is "X calls Y calls Z, and look at the mess." Style leakage edges red and the deep module dark. Sequence-style diagrams work well for "before: 6 round-trips; after: 1." If Mermaid is locally available or explicitly allowed, a Mermaid `flowchart` or `sequenceDiagram` is also acceptable, but do not make the report depend on a CDN by default.

```html
<div class="card">
  <svg viewBox="0 0 640 180" role="img" aria-label="Order flow before deepening">
    <rect x="24" y="60" width="140" height="52" rx="10" fill="#fff" stroke="#cbd5e1"/>
    <text x="94" y="91" text-anchor="middle" font-size="13">OrderHandler</text>
    <line x1="164" y1="86" x2="236" y2="86" stroke="#64748b" marker-end="url(#arrow)"/>
    <!-- continue the call flow with module boxes, seam lines, and leakage edges -->
  </svg>
</div>
```

### Hand-built boxes-and-arrows (when Mermaid's layout fights you)

Modules as `<div>`s with borders and labels. Arrows as inline SVG `<line>` or `<path>` elements positioned absolutely over a relative container. Reach for this when you want the "after" diagram to feel like one thick-bordered deep module with greyed-out internals — Auto-layout graph tools often will not render that with the right weight.

### Cross-section (good for layered shallowness)

Stack horizontal bands (`h-12 border-l-4`) to show layers a call passes through. Before: 6 thin layers each doing nothing. After: 1 thick band labelled with the consolidated responsibility.

### Mass diagram (good for "interface as wide as implementation")

Two rectangles per module — one for interface surface area, one for implementation. Before: interface rectangle is nearly as tall as the implementation rectangle (shallow). After: interface rectangle is short, implementation rectangle is tall (deep).

### Call-graph collapse

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: one accent (emerald or indigo) plus red for leakage and amber for warnings.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for module labels inside diagrams — they should read as schematic, not as UI.
- Avoid external scripts by default. The report should be static HTML, embedded CSS, and inline SVG/HTML diagrams unless the user explicitly accepts local/CDN diagram tooling.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise — but the architectural nouns and verbs come straight from [LANGUAGE.md](LANGUAGE.md). Concision is not an excuse to drift.

**Use exactly:** module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality.

**Never substitute:** component, service, unit (for module) · API, signature (for interface) · boundary (for seam) · layer, wrapper (for module, when you mean module).

**Phrasings that fit the style:**

- "Order intake module is shallow — interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in prod, in-memory in tests."

**Wins bullets** name the gain in glossary terms: *"locality: bugs concentrate in one module"*, *"leverage: one interface, N call sites"*, *"interface shrinks; implementation absorbs the wrappers"*. Don't write *"easier to maintain"* or *"cleaner code"* — those terms aren't in the glossary and don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in [LANGUAGE.md](LANGUAGE.md), reach for one that is before inventing a new one.
