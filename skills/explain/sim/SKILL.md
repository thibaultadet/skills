---
name: sim
description: Builds a self-contained interactive HTML page that teaches a technical subject step by step — one step visible at a time, a Next button, and scenarios where the reader makes choices and sees the consequences. Use when the user asks for an interactive explanation, "pas à pas", "page HTML interactive", or when another skill (e.g. catch-up) picks the interactive format for branching business rules.
metadata:
  credits:
    article: "Quatre façons d'expliquer un harnais"
    author: Taha K
    url: "https://tahak.me/posts/quatre-facons-expliquer-un-harnais/"
---

# Explain — interactive HTML

## Content rules

- 4 to 8 steps. Each step = one idea: title, ≤4 short sentences, optional code excerpt (≤10 lines) with its `file:line`.
- Order: context → mechanics → scenarios → recap of the rules.
- **Scenarios**: at least one step where the reader picks between 2–4 choices ("Le tuteur est désactivé", "Le rapport est déjà signé"…). Each choice reveals its consequence: what the code does, which branch, which `file:line`, what the user sees.
- Every scenario mirrors a real branch in the code (an `if`, a policy rule, a state transition, a validation). Read it before writing it. No invented behaviour.
- Mark wrong-intuition choices explicitly ("Piège : …") — they teach the most.
- UI copy in the user's language.

## Technical rules

- One `.html` file, inline CSS + vanilla JS, no dependency.
- One step visible at a time. Buttons "Précédent" / "Suivant", progress "3 / 7", keyboard ← →.
- "Suivant" stays enabled on scenario steps; choices are optional exploration.
- Colors as CSS variables on `:root`, dark mode via `prefers-color-scheme`. Explicit `body` background.
- Works at phone width; code blocks scroll horizontally, page does not.

## Delivery

1. Write to `<scratchpad or /tmp>/explain/<slug>.html`.
2. `open <file>`.
3. In chat: one line with the path + the list of steps (titles only).
