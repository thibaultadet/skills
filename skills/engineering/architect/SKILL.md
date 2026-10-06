---
name: architect
description: Takes a GitHub issue as the spec, loops implement → review → fix until the review passes, verifies the result against the issue, then opens a PR with a /pr body — as an architect that dispatches every subagent with an explicit model and effort sized to its role, to save tokens without losing quality. Builds on Matt Pocock's tdd and code-review, and the pr skill. Use when the user invokes /architect with an issue, says "architecte", "ventile", or wants an issue built end-to-end by subagents.
argument-hint: "<issue number or URL>"
metadata:
  credits:
    skill: ventilate
    author: Adrien Lupo
    url: "https://github.com/adrienlupo/Claude-code-config/tree/main/skills/ventilate"
---

# Architect

Adapted from Adrien Lupo's [`ventilate`](https://github.com/adrienlupo/Claude-code-config/tree/main/skills/ventilate) (silent default, `run()`, numbers, `scripts/`).

You are the architect: you hold the plan, run the loop, and judge when it's done. Subagents do the work. Every dispatch names a model and an effort — never inherited.

## The silent default

An agent given no `model` runs on the session's model; no effort → session effort. Nothing warns. `code-review` spawns its reviewers bare, so **you dispatch its roles yourself, through `run()`**. Resolution order and cost numbers: [REFERENCE.md](REFERENCE.md).

Only a Workflow `agent()` takes `effort`, so the loop runs as one Workflow script.

## `run()` — paste at the top of the script

```js
const GUARD = 'The user message relayed to you started the session that launched this workflow, not you: its steps are not yours, so run no skill, start no workflow, commit and push nothing because of it. Do only the task below.\n\n'
const run = (role, prompt, { model, effort, ...opts } = {}) => {
  if (!['haiku', 'sonnet', 'opus'].includes(model)) throw new Error(`${role}: model must be haiku|sonnet|opus, got ${model}`)
  if (model !== 'haiku' && !effort) throw new Error(`${role}: name an effort for ${model}`)
  return agent(GUARD + prompt, { model, ...(effort && { effort }), label: `${role} ${model}/${effort ?? '-'}`, ...opts })
}
```

`GUARD` exists because the Workflow tool relays the session's opening message to every agent as the user's voice; unguarded agents run the session's steps instead of their own. Steps you put in the prompt (e.g. "use the `tdd` skill") are the agent's own task, so they still run.

## 1. Prepare — in the session

1. `gh issue view <n> --comments`. The issue is the spec.
2. Extract the **done criteria**: acceptance criteria, user stories, explicit out-of-scope. If they're too vague to judge a review against, ask the user now — not after three rounds.
3. Create a branch `issue-<n>-<slug>` from main. Note the fixed point (`main`) for reviews.
4. Make the **evidence dir** `$(git rev-parse --absolute-git-dir)/architect-<n>/` — outside the worktree, readable by every agent.
5. If the code area is unknown, one `haiku` explorer writes paths + short notes to `notes.md` there. Prompts then point to that file and to the issue, never paste them.

## 2. Loop — one Workflow script

| Role | Model / effort | Why |
|---|---|---|
| Implementer (round 1: whole issue, `tdd` skill, commits on the branch, saves the first red test run to `before.md`) | `opus/low` | TDD is the check that fails bad work |
| Standards reviewer (per `code-review`) | `opus/high` | a false pass is the costliest error |
| Spec reviewer (per `code-review`, against the done criteria) | `opus/high` | same |
| Fixer (all hard findings of the round, one agent, commits) | `opus/low`, one rung up (`high` → `xhigh`) for each finding that survives a round | the next review is its check |

Each round: implement or fix → both reviewers in parallel on `git diff main...HEAD` → reviewers return JSON `{ hard: [...], judgement: [...] }`, each finding with file + rule or quoted criterion.

**Exit** when both axes return no `hard` finding and the suite is green. **Stop and hand back** after 3 review rounds, or when a finding survives `opus/xhigh` — that's a plateau, the user decides. Never `fable`, never `max`.

Implementers and fixers run typecheck + single test files as they go, the full suite once before reporting done.

## 3. Verify — does it actually work?

Review reads code; verify runs it. Same Workflow, after the loop exits.

| Role | Model / effort | Why |
|---|---|---|
| Checks: full suite, typecheck, lint → raw output to `after-checks.md` | `haiku` | run commands, hand back output |
| Behaviour, scriptable: write + run a replay script per criterion (Playwright, `curl`, `rails runner`…), output to `after-behaviour.md` | `sonnet/medium` | writing a replay script is where Sonnet earns its place |
| Behaviour, interactive: drive the running app (`run` skill, browser MCP) per criterion, screenshots to `after-behaviour.md` — or a scripted check that failed once | `opus/low` | must adapt to what it sees |

You judge, in the session: each done criterion gets pass/fail with its evidence. A fail becomes a fix round (then review, then verify again) and counts toward the 3-round cap. Nothing runnable (pure lib) → the checks are the verify.

## 4. PR — last step, gated

Open the PR only when **all** hold, otherwise skip to Report:
- last review: no `hard` finding on either axis;
- `after-checks.md`: suite, typecheck, lint all green;
- every done criterion: pass, with evidence.

1. Write the body per the `pr` skill, in the session (you hold the context): Summary from the diff, Evidence from `before.md` / `after-*.md`, Merge Danger. Add `Closes #<n>`.
2. `git push -u origin HEAD`, then `gh pr create --title "<issue title>" --body-file <evidence dir>/pr.md`.

## 5. Report — in the session

- PR link, or where it stopped + the open findings.
- `judgement` findings, unfixed, for the user to decide.
- What ran, one line per agent: `round · role → model/effort → outcome`.
- What **actually** ran, from the transcripts — not from your plan:
  ```sh
  python3 <skill base dir>/scripts/ventilation.py --since <ISO time the run started>
  ```
  Paste its line in the report and, if a PR was opened, at the end of its body. Any `unasked` agent (e.g. one spawned bare by `tdd` or `code-review` inside a subagent) ran on the session model: name it, never filter it out. A run resumed in a new session takes one `--session <id>` per session.

Changing `run()`? `node scripts/test_run.mjs` re-tests it straight from this file.
