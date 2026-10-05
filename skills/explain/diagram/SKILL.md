---
name: diagram
description: Explains a technical subject as a single SVG diagram (5 to 7 blocks max, arrows for flows) instead of text, then opens it. Use when the user asks for "un schéma", a diagram, a visual of a flow/architecture, or when another skill (e.g. catch-up) picks the diagram format for a multi-component flow.
---

# Explain — SVG diagram

## Content rules

- 5 to 7 blocks. More → merge blocks or split into two diagrams.
- A block = a component that does something (model, controller, service, job, page, external service). Title = code name; subtitle = role in ≤5 words (user's language).
- Arrows = flows only. Label each arrow with a verb or the payload ("POST /reports", "enqueue", "renvoie PDF").
- One reading direction: left→right for request flows, top→bottom for lifecycles.
- Highlight what is new/changed (accent border) vs existing (neutral border) when explaining a diff. Add a 2-entry legend in that case.
- Verify every block and arrow against the code. No guessed edges.

## Technical rules

- Standalone `.svg`, `viewBox` set, no external fonts/scripts. `font-family: system-ui`.
- Arrowheads via one `<marker>` in `<defs>`.
- Readable in light and dark: colors in an inner `<style>` with `@media (prefers-color-scheme: dark)` overrides, plus an explicit background `<rect>`.
- No overlapping arrows or text. Min 14px text.

## Delivery

1. Write to `<scratchpad or /tmp>/explain/<slug>.svg`.
2. Validate: `xmllint --noout <file>`.
3. `open <file>`.
4. In chat: one line with the path + max 3 bullets on what to look at first.
