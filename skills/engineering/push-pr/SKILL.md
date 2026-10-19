---
name: push-pr
description: Cleans the branch's commits, checks the sandcastle pre-review points, pushes it and opens (or updates) a GitHub PR with a body per the pr skill. Use when the user wants to open a PR or submit their work for review.
allowed-tools: Bash, Skill
---

Turn the current branch into a reviewable PR: a clean history, pushed, with a body written per the `pr` skill.

## 1. Gather context

- `git status`: the working tree must be clean. If it is dirty, stop and ask.
- `git branch --show-current`, `git log main..HEAD --oneline`, `git diff main...HEAD --stat`.
- `git rev-parse --abbrev-ref @{u}` succeeds → the branch is already pushed.
- `gh pr view --json number,url,isDraft` succeeds → a PR already exists; steps 4 and 5 update it.

Done when you know the commits, the changed files, whether the branch is on the remote, and whether a PR exists.

## 2. Check the pre-review points

Only when a PR exists. The **pre-review** is its latest comment whose body starts with `<!-- sandcastle-pr-check -->`:
```sh
gh api repos/{owner}/{repo}/issues/<n>/comments --paginate \
  --jq '[.[] | select(.body | startswith("<!-- sandcastle-pr-check -->"))] | last | .body'
```
No such comment → skip to step 3.

Each numbered point gets a verdict, from the code as it is now: ✅ addressed (name the commit or `file:line`) · 🔴 still open · ⚪ false positive (say why). Keep the pre-review's numbering and wording: every point stays listed, whatever its verdict. The comment itself stays as sandcastle wrote it; the verdicts go in the PR body (step 4).

Show the table to the user. Any 🔴 → ask whether to fix it first or push as is.

Done when every point has a verdict backed by code you read, and the user has decided on the 🔴 ones.

## 3. Clean the commits

Target: each commit is one logical change, and its message follows the repo's convention (read it from `git log main --oneline -20`, e.g. `type(scope): summary`). WIP, `fixup`, typo and "address review" commits fold into the commit they fix.

- History already meets the target → skip to step 4.
- Otherwise propose the target history (one line per commit: message + files) and wait for the user's go. If the branch is already pushed, say the push will need `--force-with-lease`.
- Rewrite it:
  ```sh
  before=$(git rev-parse HEAD)
  git reset --soft $(git merge-base main HEAD)
  git restore --staged .
  # per target commit, in order:
  git add <files> && git commit -m "<message>"
  ```
  A file shared by two target commits forces them into one; merge them rather than split the file.
- Verify: `git diff $before HEAD` is empty. Not empty → `git reset --hard $before` and report.

Done when the history matches the target the user approved and the tree is identical to `$before`.

## 4. Write the PR

- **Title**: under 70 chars, imperative, in English.
- **Body**: per the `pr` skill (Summary, Evidence, Merge Danger), in English. Evidence is what actually ran in this session or can run now (tests, command output); with none, the Evidence section says so. Add `Closes #<n>` when the branch or commits name an issue. With a pre-review, add a `## Pre-review` section: every point, numbered as in the comment, with its step-2 verdict. End with:
  ```
  🤖 Generated with [Claude Code](https://claude.com/claude-code)
  ```
- PR exists → carry over its `## Issues` section verbatim (`gh pr view --json body`); `take-over-pr` reads it.
- Write the body to a file (`mktemp`), so Mermaid and code blocks reach GitHub intact.

Done when the title and the body file exist.

## 5. Push and open

- `git push -u origin HEAD`, or `git push --force-with-lease` when step 3 rewrote a pushed branch.
- No PR yet → `gh pr create --title "<title>" --body-file <body file>`.
- PR exists → `gh pr edit <n> --title "<title>" --body-file <body file>`.

Done when `gh` returns the PR URL. Give it to the user.
