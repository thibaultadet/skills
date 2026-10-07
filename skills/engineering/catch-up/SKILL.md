---
name: catch-up
description: Big-picture overview, then interactive zone-by-zone walkthrough of a branch an agent (sandcastle) built, so a human stays familiar with the codebase without re-reading every changed line. Explains how the new/changed models, controllers, services etc. work, then flags where a human eye is worth it. Use after a sandcastle run, to catch up on a branch, or when the user wants a guided review of "what changed and where to look".
allowed-tools: Bash, Read, Grep, Glob, Write
---

Guide a human through a branch an agent built (typically several issues under one PRD): **the global picture first, then interactively, zone by zone**. Not a line-by-line audit — keep the human familiar with how the code works, and surface where a human eye adds value.

The user cares about, in order:

1. **Stay familiar with the architecture** — how the new/changed models, controllers, services, jobs, components work.
2. **Attention points** — questionable business logic, edge cases, security/auth, migrations/data, breaking changes, perf/N+1, missing tests.

Keep each turn short — fragment rather than dump a wall of text (see [[feedback_code_readability]]).

## Picking the explanation format

Both the global picture (step 3) and each zone (step 4) are explained with the `ste`, `diagram` or `sim` skill. Test the subject in this order; first match wins:

| Format | Skill | Pick when the subject is… |
| --- | --- | --- |
| Interactive HTML | `sim` | a **matrix**: behaviour varies along ≥2 axes (user type × status, paper × Beny case…), so "what happens if…" is the real question. |
| SVG diagram | `diagram` | a **pipeline**: ≥3 hops across components or layers (page → API client → controller → service → job), or a data relationship. |
| STE text | `ste` | neither: one model/service, a validation, a refactor, independent zones. |

Announce it in one line with the reason ("Format : schéma — le flux traverse 4 couches"). The user can override anytime ("en texte", "en schéma", "interactif"). Load the matching skill and follow its rules.

## Process

### 1. Pin the diff (silent)

```
git log main..HEAD --oneline          # commits + issue refs (#123)
git diff main...HEAD --stat           # three-dot = vs merge-base
```

Diff empty or current branch is `main` → ask which ref to review. Issue refs in commits give the intent; fetch them via `docs/agents/issue-tracker.md` only if subjects aren't self-explanatory.

### 2. Build the map (silent)

Group changed files by layer/domain (backend, frontend; skip tests, config, lockfiles, generated files). Order by importance, not `git`'s alphabetical order.

Read the changed files when the diff isn't enough — never guess behaviour from a name (see [[feedback_read_ui_components]]). Lean on `docs/architecture.md`, `docs/backend.md`, `docs/frontend.md`, `GLOSSARY.md`.

**Explore beyond the diff.** The diff shows _what changed_, not _what it relies on_ — that's where real risks hide. Open what the diff calls: the Rails action a new API client hits (authorization, params, side effects), callers and callees of a changed method. Can't access a file you need → say so ("controller not read"), don't guess.

Decide the **zone order**: biggest architectural impact first, trivial last. A zone is a coherent unit (model + its service, controller + serializer, one front feature), not one file.

### 3. Global picture — before any zone

Whole before parts: entering a zone, the user must already know where it sits in the flow and why it exists. Explain the branch **as a whole**, in the format picked above (the whole branch is the subject — often `diagram` when zones interact):

- **End to end** — the problem solved, the main flow from entry point to outcome, which existing parts of the app it plugs into.
- **Key concepts** — new domain terms, models, states or rules the zones will refer to (max ~5). Skip if none.
- **How the issues fit** — when several: what each adds (foundation → feature → polish).

Then, as text in chat, the **zone list** in walkthrough order, one line each, tied to its role in the flow ("Zone 2 — the job that sends the email triggered at step 3").

Stay high-level: no `file:line`, no attention points, no inner mechanics. Stop: answer big-picture questions, then start zone 1 (or reorder).

### 4. Walkthrough — one zone per turn

- **How it works** — in the picked format: what the model/controller/service does, how it fits the existing architecture, which Rails/Next conventions it follows.
- **Attention points — always as text in chat** (they need clickable `file:line`). **Verified, never speculated**: only what you confirmed by reading the code. Each: _where_ (file:line), _why_ a human eye helps, the concrete _risk_. Tag: `🔴 business` · `🔒 security/auth` · `🗃️ migration/data` · `💥 breaking` · `⚡ perf/N+1` · `🧪 tests` or other fitting tag.
- **Don't manufacture risk.** No point in a zone → say so in one line. Can't name the concrete failure → drop it.
- **Unverified = question**, not finding ("worth confirming the controller authorizes X").

**Then stop.** Dig deeper, or next zone? Never dump every zone at once.

### 5. Final summary

Once every zone is covered (or the user wraps up): **one list** of retained attention points, ranked by risk, each with clickable `file:line` and one sentence — the "look here before merging" checklist.

## Principles

- **Whole before parts.** Never open a zone before the user has the end-to-end flow in mind.
- **Selective, not exhaustive.** Ignore whitespace, trivial renames, generated files, lockfiles.
- **Explain while teaching.** Relate changes to existing repo patterns, name the conventions.
- **Interactive = you wait.** One step, one pause.
- **Honest about risk.** "Nothing to flag here" is a valid answer.
