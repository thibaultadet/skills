---
name: review-changes
description: Reviews current git changes (staged and unstaged) for code quality, patterns, and potential issues. Use when the user asks to review, audit, or check their current changes before committing.
allowed-tools: Bash, Read, Grep, Glob
---

Review the current git changes in this repository.

1. Run `git diff HEAD` to see all unstaged and staged changes
2. Run `git status` to understand which files are affected
3. For each changed file, read the relevant context if needed

Then provide a concise review focusing on:

- **Correctness**: logical errors, edge cases, missing validations
- **Consistency**: does the code follow patterns already established in the codebase (Rails conventions, Next.js patterns, French locale, etc.)
- **Simplicity**: unnecessary complexity, premature abstractions, dead code
- **Security**: auth checks, input validation, SQL injection, XSS risks
- **Potential issues**: anything that could break in production

Format your review by file, keeping feedback short and actionable. Skip files with trivial changes (whitespace, comments). End with a short overall summary.
