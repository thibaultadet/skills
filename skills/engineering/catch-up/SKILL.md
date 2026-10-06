---
name: catch-up
description: Interactive, zone-by-zone walkthrough of a branch an agent (sandcastle) built, so a human stays familiar with the codebase without re-reading every changed line. Explains how the new/changed models, controllers, services etc. work, then flags where a human eye is worth it. Use after a sandcastle run, to catch up on a branch, or when the user wants a guided review of "what changed and where to look".
allowed-tools: Bash, Read, Grep, Glob, Write
---

Guide a human through a branch an agent built (typically several issues under one PRD), **interactively, zone by zone**. The goal is not a full line-by-line audit — it is to keep the human familiar with how the code works and to surface the spots where a human eye actually adds value.

Two things the user cares about, in this order:

1. **Stay familiar with the architecture** — how do the new/changed models, controllers, services, jobs, components actually work?
2. **Attention points** — where should a human look closely? (questionable business logic, edge cases, security/auth, migrations/data, breaking changes, perf/N+1, missing tests)

Keep each turn short and readable — fragment rather than dumping a wall of text (see [[feedback_code_readability]]).

## Process

### 1. Pin the diff (silent)

Scope = everything the branch changed since it diverged from `main`.

```
git log main..HEAD --oneline          # commits + issue refs (#123)
git diff main...HEAD --stat           # three-dot = vs merge-base
```

Confirm the diff is non-empty. If the current branch _is_ `main`, ask the user which branch/ref to review.

Note the issue references (`#123`, `Closes #45`) from the commit messages — they tell you the intent. Fetch them via `docs/agents/issue-tracker.md` only if the commit subjects aren't self-explanatory.

### 2. Build the map (silent)

Group the changed files by layer/domain — don't show this raw. You're building your own mental model so the walkthrough is ordered by importance, not by `git`'s alphabetical order.

- Backend: models, controllers, services, jobs, serializers, migrations
- Frontend: components, hooks, pages/routes, utils
- Tests, config, generated/lockfiles (→ skip these in the walkthrough)

Read the actual changed files where the diff alone isn't enough to explain mechanics — never guess a component's behavior from its name (see [[feedback_read_ui_components]]). Lean on `docs/architecture.md`, `docs/backend.md`, `docs/frontend.md`, `GLOSSARY.md` for domain vocabulary.

**Explore beyond the diff.** The diff shows *what changed*, not *what the change relies on*. A file being outside the diff is not a reason to skip it — it's the main reason a real risk gets missed. When behaviour or a risk depends on code the diff only *calls*, open it: the Rails controller/action a new front API client hits (check authorization, params, side effects), the callers and callees of a changed method/component. Use `git grep`, `Read`, `Grep` freely. If you can't access a file you'd need to judge something, say so explicitly ("controller not read") rather than guessing.

Decide the **zone order**: biggest architectural impact first (new model / changed domain flow), trivial last. A "zone" is a coherent unit — a model + its service, one controller + its serializer, one feature on the front — not one file.

### 3. Orientation (one short message)

Before diving, give the user:

- The branch's **overall intent** in 2-3 lines, and how the issues relate to each other.
- The **list of zones** you'll walk through, in order, one line each.
- **Overview diagram, only when it helps**: the zones interact (a front feature calling a new endpoint that enqueues a job…), so their links are hard to hold from a list. Skip it when zones are independent or there are fewer than 3. Use the `diagram` skill, one block per zone.

Then stop and let them start (or reorder).

### 4. Walkthrough — one zone per turn

For each zone, first **pick the format** of the "How it works" part. Test the zone's mechanics in this order; the first match wins:

| Format | Skill | Pick when the zone is… |
| --- | --- | --- |
| Interactive HTML | `sim` | a **matrix**: the behaviour varies along ≥2 axes (user type × status, paper × Beny case, open request or not…), so "what happens if…" is the real question. |
| SVG diagram | `diagram` | a **pipeline**: the mechanics read as ≥3 numbered hops that cross components or layers (event → callback → `log_event` → JSON → sort; page → API client → controller → service → job), or a data relationship. |
| STE text | `ste` | neither: one model/service, a validation, a refactor. |

Announce it in one line with the reason ("Format : schéma — le flux traverse 4 couches"). The user can override at any time ("en texte", "en schéma", "interactif"). Load the matching skill and follow its rules.

Then, for each zone:

- **How it works** — in the chosen format. The mechanics: what the model/controller/service does, how it fits into the existing architecture, which Rails/Next conventions it follows. This is the "stay familiar" part.
- **Attention points — always as text in chat**, whatever the format above (they need clickable `file:line`). **Verified, never speculated.** A point earns its place only if you confirmed it by reading the code. For each: _where_ (file:line), _why_ a human eye helps, and the concrete _risk_. Tag each one: `🔴 business` · `🔒 security/auth` · `🗃️ migration/data` · `💥 breaking` · `⚡ perf/N+1` · `🧪 tests` or any other appropriate tags.
- **Don't manufacture risk to fill space.** If a zone has no attention point, say so in one line and move on. If you can't name the concrete failure a point would cause, it's not an attention point — drop it.
- **Don't dress up the unverified as a finding.** Something you couldn't check is a _question_ for the user ("worth confirming the controller authorizes X"), not a flagged defect.

**Then stop.** Ask: dig deeper into this zone, or move to the next one? Let the user ask questions before advancing. Never dump every zone at once.

### 5. Final summary

Once every zone has been covered (or the user asks to wrap up), produce **a single list** of the retained attention points, ranked by risk (most critical first), each with a clickable `file:line` and one sentence. This is the "where a human must look before merging" checklist.

## Principles

- **Selective, not exhaustive.** The point is to stay familiar and target the risks, not to re-read everything. Ignore whitespace, trivial renames, generated files, lockfiles.
- **Explain while teaching.** The user wants to keep learning the architecture — relate changes to existing patterns in the repo, name the conventions.
- **Interactive = you wait.** One zone, one pause. No 2000-word monologue.
- **Honest about risk.** "Nothing to flag here" is a valid and useful answer.
