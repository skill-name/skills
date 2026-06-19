# HTML Report Format

The file review is rendered as a single self-contained HTML file in the OS temp directory. Tailwind and Mermaid both come from CDNs. Mermaid handles graph-shaped diagrams reliably; hand-built divs, inline SVG, and CSS layout handle the more editorial visuals (file dissection, cohesion maps, flow diagrams). Mix the two — don't lean on Mermaid for everything, it'll start to look generic.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>File review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* custom layer for visual patterns the file vocabulary needs */
      .fissure { border-left: 4px solid #dc2626; }
      .crack  { border-left: 4px dashed #2563eb; }
      .cohesion-hot { background: repeating-linear-gradient(45deg, #fef3c7 0px, #fef3c7 8px, #fff 8px, #fff 16px); }
      .gravity { background: #fee2e2; border-radius: 0.25rem; }
      .after-inline { opacity: 0.4; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-pick">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, plus a compact legend:
- Coloured left border = fissure indicator
- Hatched background = cohesion gap
- Red highlight = gravity hotspot
- Dashed blue border = crack
- Faded text = to be inlined or removed

No introduction paragraph — straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the file vocabulary without ceremony.

Each candidate is one `<article>`:

- **Title** — short, names the fissure and file (e.g. "Responsibility drift in `order-service.ts`").
- **Badge row** — effort badge (`Quick` = emerald, `Moderate` = amber, `Larger` = slate), plus a fissure type tag (responsibility-drift, cohesion-gap, gravity-hotspot, flow-break, surface-mismatch, weight-imbalance, crack-ignored).
- **File** — monospaced path, `font-mono text-sm`.
- **Responsibility** — what the file claims to do.
- **Before / After diagram** — the centrepiece. Two columns, side by side. See patterns below.
- **Fissure** — one sentence. What hurts.
- **Why it matters** — one sentence. What goes wrong.
- **Fix** — one sentence. What changes.
- **Expected effect** — bullets, ≤6 words each. e.g. "Cohesion up", "Weight down to 80 loc", "Flow reads top-down", "Gravity spreads across 2 files".

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point.

### File dissection (the workhorse for cohesion, weight, cracks)

Show the file as a vertical stack of labelled blocks, each representing a logical section (imports, types, function A, function B, helpers, etc.). Fissured sections get coloured backgrounds.

**Before**: blocks are different sizes, one block is massive (gravity), and unrelated blocks sit together.
**After**: same blocks rearranged, the massive block is split, unrelated blocks moved to a new dotted-outline column labelled with the new file name.

```html
<div class="grid grid-cols-2 gap-4">
  <div>
    <div class="text-xs uppercase tracking-wider text-slate-500 mb-1">Before</div>
    <div class="border border-slate-200 rounded p-2 space-y-1">
      <div class="h-6 bg-slate-100 rounded text-xs flex items-center px-2">imports</div>
      <div class="h-8 bg-slate-100 rounded text-xs flex items-center px-2">types</div>
      <div class="h-24 bg-red-100 border border-red-300 rounded text-xs flex items-center px-2 font-bold">validateOrder() — 120 lines</div>
      <div class="h-6 bg-slate-100 rounded text-xs flex items-center px-2">formatAddress()</div>
      <div class="h-6 bg-amber-100 rounded text-xs flex items-center px-2">sendEmail() ← unrelated</div>
    </div>
  </div>
  <div>
    <div class="text-xs uppercase tracking-wider text-slate-500 mb-1">After</div>
    <div class="border border-slate-200 rounded p-2 space-y-1">
      <div class="h-6 bg-slate-100 rounded text-xs flex items-center px-2">imports</div>
      <div class="h-8 bg-slate-100 rounded text-xs flex items-center px-2">types</div>
      <div class="h-10 bg-green-100 border border-green-300 rounded text-xs flex items-center px-2">validateOrder() — extracted</div>
      <div class="h-6 bg-slate-100 rounded text-xs flex items-center px-2">formatAddress()</div>
    </div>
    <div class="mt-2">
      <div class="text-xs uppercase tracking-wider text-slate-500 mb-1">→ moved to <code>notifications.ts</code></div>
      <div class="border border-dashed border-blue-400 rounded p-2 bg-blue-50 space-y-1">
        <div class="h-6 bg-white rounded text-xs flex items-center px-2">sendEmail()</div>
      </div>
    </div>
  </div>
</div>
```

### Gravity heatmap (good for "one monster section")

Render the file as a horizontal bar chart where section width scales with lines of code. Colour from green (small) through amber to red (large). The red section is the gravity hotspot.

**Before**: one enormous red bar dwarfing everything.
**After**: the red bar is broken into several amber/green bars, each labelled with the extracted function name.

### Flow arrows (good for flow breaks, forward references)

Show function definitions as boxes with their line numbers down the side. Overlay arrows showing where calls go:

**Before**: arrows zigzag upward (definition after use), cross between unrelated sections.
**After**: all arrows go downward — file reads top to bottom.

```html
<div class="grid grid-cols-2 gap-4">
  <div>
    <div class="text-xs uppercase tracking-wider text-slate-500 mb-1">Before</div>
    <svg viewBox="0 0 200 200" class="w-full">
      <!-- boxes with line numbers -->
      <rect x="10" y="10" width="80" height="30" rx="4" fill="#e2e8f0" />
      <text x="14" y="29" font-size="10" fill="#475569">useConfig()</text>
      <text x="96" y="29" font-size="8" fill="#94a3b8">L42</text>
      <rect x="40" y="60" width="100" height="30" rx="4" fill="#e2e8f0" />
      <text x="44" y="79" font-size="10" fill="#475569">transformData()</text>
      <text x="146" y="79" font-size="8" fill="#94a3b8">L15</text>
      <rect x="10" y="120" width="120" height="30" rx="4" fill="#e2e8f0" />
      <text x="14" y="139" font-size="10" fill="#475569">parseInput()</text>
      <text x="136" y="139" font-size="8" fill="#94a3b8">L10</text>
      <!-- arrow up — flow break -->
      <path d="M95,40 L95,50 L140,50 L140,60" fill="none" stroke="#dc2626" stroke-width="2" marker-end="url(#arrow-red)" />
      <path d="M130,90 L130,100 L70,100 L70,120" fill="none" stroke="#dc2626" stroke-width="2" marker-end="url(#arrow-red)" />
    </svg>
  </div>
  <div>
    <div class="text-xs uppercase tracking-wider text-slate-500 mb-1">After</div>
    <svg viewBox="0 0 200 200" class="w-full">
      <rect x="10" y="10" width="100" height="30" rx="4" fill="#e2e8f0" />
      <text x="14" y="29" font-size="10" fill="#475569">parseInput()</text>
      <text x="116" y="29" font-size="8" fill="#94a3b8">L10</text>
      <rect x="40" y="60" width="100" height="30" rx="4" fill="#e2e8f0" />
      <text x="44" y="79" font-size="10" fill="#475569">transformData()</text>
      <text x="146" y="79" font-size="8" fill="#94a3b8">L15</text>
      <rect x="10" y="120" width="80" height="30" rx="4" fill="#e2e8f0" />
      <text x="14" y="139" font-size="10" fill="#475569">useConfig()</text>
      <text x="96" y="139" font-size="8" fill="#94a3b8">L42</text>
      <!-- arrows down — clean flow -->
      <path d="M60,40 L60,60" fill="none" stroke="#22c55e" stroke-width="2" marker-end="url(#arrow-green)" />
      <path d="M90,90 L90,120" fill="none" stroke="#22c55e" stroke-width="2" marker-end="url(#arrow-green)" />
    </svg>
    <figcaption>After: definitions before usage — file reads top-down.</figcaption>
  </div>
</div>
```

Be sure to include the arrowhead markers in a `<defs>` block:
```html
<defs>
  <marker id="arrow-red" viewBox="0 0 10 10" refX="10" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,5 L0,10" fill="#dc2626"/></marker>
  <marker id="arrow-green" viewBox="0 0 10 10" refX="10" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,5 L0,10" fill="#22c55e"/></marker>
</defs>
```

### Surface vs. internals (good for surface mismatch)

Two stacked column charts:

**Before**: public exports column is taller than internal logic column — the surface is bigger than what it abstracts.
**After**: public exports column is short, internal logic column is tall — the file earns its surface.

### Mermaid flowchart (for dependency structure within/across files)

Use a Mermaid `flowchart` when the point is "this file imports N things it shouldn't" or "extracting X would clean the dependency graph."

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[order-service.ts] --> B[email.ts]
      A --> C[pdf.ts]
      A --> D[user.ts]
      D -.-> A
      classDef fissure stroke:#dc2626,stroke-width:2px;
      class A fissure
  </pre>
</div>
```

### File weight bar (good for weight imbalance)

A single horizontal bar per file, scaled to lines of code. The target file is one enormous bar next to peer files at 10% its size. After: the bar is split proportionally across two or three files.

```html
<div class="space-y-2">
  <div class="text-xs uppercase tracking-wider text-slate-500">Before</div>
  <div class="h-6 bg-red-200 border border-red-400 rounded" style="width: 100%" title="utils.ts — 600 loc"></div>
  <div class="h-6 bg-stone-200 border border-stone-300 rounded" style="width: 12%" title="validator.ts — 72 loc"></div>
  <div class="h-6 bg-stone-200 border border-stone-300 rounded" style="width: 8%" title="formatter.ts — 48 loc"></div>
  <div class="text-xs uppercase tracking-wider text-slate-500 mt-4">After</div>
  <div class="h-6 bg-green-200 border border-green-400 rounded" style="width: 30%" title="order-utils.ts — 180 loc"></div>
  <div class="h-6 bg-green-200 border border-green-400 rounded" style="width: 20%" title="payment-utils.ts — 120 loc"></div>
  <div class="h-6 bg-green-200 border border-green-400 rounded" style="width: 18%" title="notification-utils.ts — 108 loc"></div>
  <div class="h-6 bg-stone-200 border border-stone-300 rounded" style="width: 12%" title="validator.ts — 72 loc"></div>
  <div class="h-6 bg-stone-200 border border-stone-300 rounded" style="width: 8%" title="formatter.ts — 48 loc"></div>
</div>
```

### Cohesion zigzag (good for interwoven concerns)

Two horizontal bands showing which lines belong to which concern:

**Before**: red and blue segments alternate chaotically — concerns are interleaved.
**After**: all red lines, then all blue lines, optionally in separate files.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: emerald for fixed/quick, amber for moderate, red for fissures, blue for cracks, stone for unchanged.
- Keep diagrams ~360px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for section labels inside diagrams — they should read as schematic, not as UI.
- Include arrowhead SVG definitions once in a `<defs>` block near the top of the body. Reuse them across diagrams.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. The report is otherwise static — no app code, no interactivity beyond Mermaid's own rendering.

## Top pick section

One larger card. File name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise — but the structural nouns and verbs come from the file vocabulary. Concision is not an excuse to drift.

**Use exactly:** file, responsibility, cohesion, surface, gravity, flow, crack, weight.

**Never substitute:** component, service, unit (for file) · API, signature (for surface) · boundary, seam (for crack) · spaghetti, mess (for gravity/cohesion gap).

**Phrasings that fit the style:**

- "`handlers.ts` has a cohesion gap — email logic sits next to order validation."
- "Gravity concentrates in `processPayment()`: 180 lines, four responsibilities."
- "Flow breaks: `parseInput()` is called on line 90 but defined on line 200."
- "Surface mismatch: 8 public exports for 3 internal functions."
- "Weight imbalance: `utils.ts` is 600 lines; peer files average 60."
- "Crack ignored: the three `format*` functions belong together."

**Expected-effect bullets** name the gain in file vocabulary terms: *"cohesion: email concern moves to its own file"*, *"gravity: 180-line function splits into 6 focused helpers"*, *"flow: file reads top-down"*, *"weight: 600 → 180+120+108"*. Don't write *"cleaner code"* or *"easier to maintain"* — those don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in the file vocabulary, reach for one that is before inventing a new one.
