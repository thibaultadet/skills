---
name: take-over-pr
description: Take over a PR, typically one a sandcastle run opened. Rebase it on main, resolve conflicts, then do a quick review against the spec and the pre-review points.
disable-model-invocation: true
---

Take ownership of a PR: bring it up to date with `main`, then run a **quick review** that answers one question: can this merge? A full standards audit belongs to `/code-review`.

The argument is a PR number. If there is none, use the PR of the current branch (`gh pr view`).

## 1. Check out the PR

- Require a clean working tree. If it is dirty, stop and ask.
- `gh pr checkout <n>`, then `git fetch origin main`.
- Note the PR state (`gh pr view <n> --json state,isDraft,mergeable,statusCheckRollup`). If the PR is merged or closed, stop and report. A draft PR skips CI: its checks read `SKIPPED`, not green.

Done when HEAD is the PR head branch and `origin/main` is fresh.

## 2. Rebase on main if behind

- `git merge-base --is-ancestor origin/main HEAD` succeeds → already up to date. Skip to step 3.
- Otherwise run `git rebase origin/main`. On conflict, invoke the `resolving-merge-conflicts` skill. The PR's intent comes from its body and linked issue. `main`'s intent comes from the commits that landed since the branch diverged.
- After a rebase, run the checks for the areas the PR touches:
  - `api/` changed: rubocop on the changed Ruby files only, then `bundle exec rspec`:
    `cd api && git diff --name-only --diff-filter=d --relative origin/main...HEAD -- '*.rb' '*.rake' '*.jb' Gemfile | xargs -r bundle exec rubocop --force-exclusion`
    Keep the `xargs -r` form: an empty file list must skip rubocop, not lint the whole app. Scope rspec to the specs of the changed code and of the code that caused a conflict. The whole suite runs in CI.
  - `web/` changed: `cd web && bun run lint`.
- Leave the rebased branch local. Pushing is the user's call.

Done when the branch contains `origin/main` and the checks are green. If a check stays red after one honest fix attempt, stop and report it.

## 3. Gather the review inputs

**Spec**: the issues the PR implements. Look in the `## Issues` section of the PR body, in `closingIssuesReferences`, and in the branch name (`issue/<n>`). Read each issue with `gh issue view <n> --json title,state,body,comments` (plain `--comments` can print nothing), and follow its parent PRD when it names one. An `## Interface` section in an issue is a contract (see `docs/coding-standards.md`).

**Siblings**: the other issues of that PRD own the work this PR leaves out. Find the parent with `gh api 'repos/{owner}/{repo}/issues/<n>/parent' --jq .number` (404: none) or the PRD the issue names, then list its sub-issues:
```sh
gh api 'repos/{owner}/{repo}/issues/<prd>/sub_issues' --jq '.[] | "#\(.number) [\(.state)] \(.title)"'
```
Read the body of every sibling whose title touches what the PR does.

**Mockup**: the design the issue or PRD names (a mockup file, a design artifact link, a board the repo's design index maps to the issue). The repo's verification skill says how to render it next to the app.

**Pre-review**: the latest PR comment whose body starts with `<!-- sandcastle-pr-check -->`. Its numbered points are the attention points. Its `Manual:` line lists checks only a human can run.

If the spec or the pre-review is missing, say so and review with the one you have.

## 4. Quick review

Read the diff (`git diff origin/main...HEAD`) with both inputs next to it. Give every item a verdict:

- **Each acceptance criterion / interface item**: ✅ implemented (cite `file:line`, plus the spec covering it) · ⚠️ partial · ❌ missing.
- **Each pre-review point**, checked against the code as it is now (after the rebase): 🔴 still true · ✅ fixed · ⚪ false positive (say why).
- **The mockup**, when the spec names one: each screen the PR changes, side by side with its mockup → ✅ matches · ⚠️ differences (list each one).
- Anything the rebase changed in behaviour, even if both sides were fine alone.

Every ⚠️ and ❌ names its owner: **this PR**, or the sibling issue that covers it (cite `#n`). Propose a new issue only for a gap no sibling covers.

Done when every criterion, every pre-review point and every mockup screen has a verdict backed by code or screenshots you saw, not by the pre-review's own wording, and every gap has an owner.

## 5. Report

Keep it terse, in the terminal:

1. Rebase: `up to date` · `rebased locally, N conflicts resolved (files), push pending` · `blocked (why)`.
   CI: `green` · `red (checks)` · `skipped: draft → gh pr ready <n>`.
2. Verdict: **ready to merge** · **needs fixes** · **blocked**.
3. Table: spec items, mockup screens, then pre-review points, with their verdicts and owners.
4. Manual checks still open, copied from the pre-review's `Manual:` line.

The terminal is the only output. Leave fixes, the push, PR comments and the merge to the user.
