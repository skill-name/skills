---
name: improve-file
description: Scan a codebase for files that need structural improvement, present them as a visual HTML report, then grill through whichever one you pick.
disable-model-invocation: true
---

# Improve File

Surface **file-level fissures** — structural problems in individual source files that hurt readability, testability, and AI-navigability. Each fissure is a candidate for a targeted refactor: rename, split, reorder, inline, extract.

This command uses a **file vocabulary** built on shared concepts:

| Term | Meaning |
|------|---------|
| **File** | A single source file and its public surface |
| **Responsibility** | The single concern a file owns — one answer to "what does this file do?" |
| **Cohesion** | How tightly every line in the file relates to its stated responsibility |
| **Surface** | The public exports / API of a file — what other files depend on |
| **Gravity** | The concentration of complexity inside a file — e.g. one 200-line function, or five interwoven concerns |
| **Flow** | The reading order within a file — top-down narrative vs. scattered definitions |
| **Crack** | A natural seam for extraction — a group of functions/types that could be their own file |
| **Weight** | File size / cognitive load relative to its responsibility |

These terms replace generic talk about "clean code," "best practices," or "readability." Every suggestion names what shifts: cohesion up, weight down, gravity concentrated, flow clear.

## Process

### 1. Explore

Read the file. Don't chase heuristics rigidly — read it as a human would, and note where you experience friction:

- **Responsibility drift** — does the file do more than one thing? Does the name describe everything inside?
- **Cohesion gap** — are there blocks of code that don't relate to each other? Can you delete half the file and the other half still makes sense?
- **Gravity hotspot** — where does complexity pile up? A single monster function? An `if` forest? Duplicated logic spread across the file?
- **Flow break** — does reading the file require jumping back and forth? Are definitions after usage? Is import order hiding what the file needs?
- **Surface mismatch** — is the public API bigger than the internal logic justifies? Are internal helpers leaked as exports?
- **Weight imbalance** — is the file 600 lines when its responsibility is "validate an email address"?
- **Crack ignored** — is there a natural extraction point that wasn't taken (e.g. a block of helper functions that clearly belong together)?

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/file-review-<timestamp>.html` so each run gets a fresh file. Open it for the user — `xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows — and tell them the absolute path.

The report uses **Tailwind via CDN** for layout and styling, and **Mermaid via CDN** for diagrams where a graph/flow/sequence reliably communicates the structure. Mix Mermaid with hand-crafted CSS/SVG visuals — use Mermaid when relationships are graph-shaped (dependency flows, call chains), and hand-built divs/SVG when you want something more editorial (file dissection, cohesion maps, flow diagrams).

For each candidate, render a card with:

- **File** — the file path
- **Responsibility** — what the file claims to do (from its name or exports)
- **Fissure** — what hurts: responsibility drift, cohesion gap, gravity hotspot, flow break, surface mismatch, weight imbalance, or crack ignored
- **Why it matters** — plain English: what goes wrong when this fissure is left alone
- **Fix** — plain English: rename, split, reorder, inline, extract, or a combination
- **Before / After diagram** — side-by-side, custom-drawn, illustrating the file's structure and the fix
- **Effort** — one of `Quick`, `Moderate`, `Larger`, rendered as a badge

End the report with a **Top pick** section: which file you'd fix first and why.

**Use the file vocabulary exactly.** Don't drift into "messy," "spaghetti code," "not clean." Say "gravity hotspot," "cohesion gap," "crack ignored."

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.

Do NOT propose implementations yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, run the `/grilling` skill to walk the design tree with them — what responsibility stays, what moves, where the crack is, what the flow should be.

Side effects happen inline as decisions crystallize:

- **Extracting to a new file?** Create it immediately with a minimal scaffold and wire the import.
- **Renaming a file?** Do it in place — update the name and all imports.
- **User rejects the candidate with a load-bearing reason?** Offer to record it as an ADR (see `/domain-modeling` skill, ADR format), framed as: _"Want me to record this as an ADR so future file reviews don't re-suggest the same change?"_ Only offer when the reason would actually be needed — skip ephemeral ("not worth it right now") and self-evident reasons.
- **Fix is straightforward with no meaningful design forks?** Just do it — no need to grill. The grilling loop is for cases with real tradeoffs.
