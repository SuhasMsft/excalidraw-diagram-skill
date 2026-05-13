---
name: excalidraw-diagram
description: "Create Excalidraw diagram JSON files that make visual arguments. Workflows, architectures, concepts. Trigger: 'draw diagram', 'visualize architecture', 'dependency graph', 'flow diagram'."
---

# Excalidraw Diagram Creator

## When to Use

- "draw diagram", "visualize architecture", "flow diagram"
- "dependency graph", "sequence diagram", "system diagram"
- User wants to visualize workflows, architectures, or concepts
- NOT for data charts (use excalidraw-exec-charts) or plan diagrams (use visualize)

## Core Philosophy

**Diagrams should ARGUE, not DISPLAY.** A diagram is a visual argument showing relationships and causality that words alone can't express.

- **Isomorphism Test**: Remove all text — does the structure alone communicate the concept?
- **Education Test**: Could someone learn something concrete from this diagram?

## Process

1. **Design** — determine the visual argument, choose layout pattern
2. **Generate** — produce Excalidraw JSON with proper shapes, colors, arrows
3. **Render** — `cd .claude/skills/excalidraw-diagram/references && uv run python render_excalidraw.py <file>`
4. **Verify** — view PNG, check for overlaps, clipping, alignment issues
5. **Iterate** — fix and re-render (2-4 iterations typical)

## Shape Meaning

| Concept | Shape |
|---------|-------|
| Process / action | `rectangle` |
| Start / end / milestone | `ellipse` |
| Decision / condition | `diamond` |
| Labels / annotations | free-floating `text` |

## Rendering Defaults

- `roughness: 0` — crisp edges
- `strokeWidth: 2` — standard for shapes
- `opacity: 100` — full opacity, use color for hierarchy
- Colors from `references/color-palette.md`

## Text Rules (CRITICAL)

Text is the #1 source of rendering bugs. Follow these rules strictly:

### No Text Overwrite
- **Always calculate box dimensions BEFORE placing text.** A box must be tall enough to contain all text lines.
- **Formula:** `boxHeight = (lineCount × fontSize × lineHeight) + 2 × padding`. Use padding ≥ 8px.
- **Never stack text elements at overlapping y-coordinates.** If a box has a title (fontSize 13) and subtitle (fontSize 11), space them: `subtitle.y = title.y + (title.fontSize × title.lineHeight) + 4`.
- **Multi-line text:** Count `\n` characters. Each line adds `fontSize × lineHeight` pixels.

### Always Center Text
- **`textAlign: "center"`** for all text inside boxes — never "left" unless the diagram is a code listing.
- **Horizontal centering:** `text.x = box.x + (box.width - text.width) / 2`
- **Vertical centering (CRITICAL — most common bug):**
  ```
  textTotalHeight = lineCount × fontSize × lineHeight   (lineHeight default = 1.25)
  text.y = box.y + (box.height - textTotalHeight) / 2
  ```
  **NEVER set `text.y = box.y`** — this pins text to the top of the box. Always compute the vertical offset.
  Example: box.y=200, box.height=65, fontSize=14, 2 lines → textHeight = 2×14×1.25 = 35 → text.y = 200 + (65-35)/2 = **215** (not 200).

### Font Size Consistency
- **Same-level elements must share the same fontSize.** If partition labels are fontSize 13, ALL partition labels must be 13.
- **Hierarchy:** Title 24-26 → Section 16-18 → Label 13 → Sublabel 11-12 → Footnote 10.
- **Never mix font sizes within a visual group** (e.g., all boxes in a column should use matching sizes).

### Standalone Text (headless renderer compatibility)
- **`containerId: null`** — bound text does NOT render in headless mode.
- **`boundElements: null`** — prevent phantom bindings.
- All text must be free-floating standalone elements positioned manually.

## Arrow Rules

- **`roundness: {"type": 2}`** on ALL arrows for smooth curves.
- **Route arrows around boxes** — never let arrows cross through box interiors.
- **Label arrows with standalone text** positioned near the midpoint of the curve.

## Visual Patterns

Fan-out, convergence, tree, timeline, layered stack, pipeline, hub-and-spoke, swimlane, comparison columns, state machine, **sequence diagram**.

### Sequence Diagram Pattern
For time-ordered interactions between actors/components:

**Layout:**
- **Actor boxes** at the top in a horizontal row, evenly spaced (width 160-200, height 50-60)
- **Lifelines**: Dashed vertical lines (`strokeStyle: "dashed"`, `strokeWidth: 1`) extending from each actor box's bottom center to the diagram bottom
- **Messages**: Horizontal or slightly angled arrows between lifelines, stacked top-to-bottom in chronological order
- **Spacing**: 50-60px vertical gap between each message row
- **Labels**: Message text positioned above or beside each arrow, fontSize 12-13

**Construction rules:**
1. Actors are rectangles at y=30-50, all same height, evenly distributed across x-axis
2. Each lifeline is a line element (not arrow): dashed, from actor bottom-center straight down
3. Messages are arrows from one lifeline x to another lifeline x, at incrementing y values
4. Self-calls: arrow loops back to the same lifeline with a small rightward bump
5. Return messages: dashed arrows going right-to-left
6. Activation boxes: thin tall rectangles (width 16-20) centered on lifeline at the active y-range
7. Notes/annotations: rectangles with light yellow background positioned to the right of the relevant message

**Colors:**
- Actor boxes: colored backgrounds matching their role (blue for device components, orange for cloud)
- Lifelines: `#94a3b8` (gray)
- Forward messages: `#1e3a5f` (navy) solid arrows
- Return/response messages: `#047857` (green) dashed arrows
- Failure paths: `#dc2626` (red)

## Related Skills

- **excalidraw-exec-charts** — for data charts (bars, tables, matrices)
- **visualize** — for planning-specific diagrams (Gantt, dependency, heatmap)
