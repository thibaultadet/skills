---
name: push-pr
description: Pushes the current branch to GitHub and creates a pull request with a generated description. Use when the user wants to open a PR, push and create a PR, or submit their work for review.
allowed-tools: Bash
---

Push the current branch and create a pull request on GitHub.

## Steps

1. Run these commands in parallel to gather context:
   - `git status` — check for uncommitted changes
   - `git branch --show-current` — get the current branch name
   - `git log main...HEAD --oneline` — list all commits since diverging from main
   - `git diff main...HEAD --stat` — list changed files

2. If there are uncommitted changes, warn the user and stop — do not proceed without a clean working tree.

3. Analyze the commits and diffs to draft:
   - A short PR **title** (under 70 chars, imperative form, in English)
   - A PR **body** with this structure (in English):
     ```
     ## Context
     <Why this change? What problem does it solve?>

     ## Changes
     <List of main changes>

     ## Test Plan
     - [ ] <Manual or automated test to perform>

     🤖 Generated with [Claude Code](https://claude.ai/claude-code)
     ```

4. Push the branch with `git push -u origin HEAD`.

5. Create the PR with:
   ```
   gh pr create --title "<title>" --body "<body>"
   ```

6. Return the PR URL to the user.
