# Skills

Agent skills I use with Claude Code.

## Install

```sh
npx skills add thibaultadet/skills
```

Or as Claude Code plugins:

```sh
claude plugin marketplace add thibaultadet/skills
claude plugin install thibaultadet-skills@thibaultadet
```

## Skills

### Engineering

- **architect** — from a GitHub issue: implement → review → fix loop, verify, then PR — subagents sized per role (model/effort) to save tokens. Adapted from [Adrien Lupo's `ventilate`](https://github.com/adrienlupo/Claude-code-config/tree/main/skills/ventilate).
- **catch-up** — interactive zone-by-zone walkthrough of an agent-built branch, with attention points.
- **push-pr** — clean the commits, check the pre-review points, push the branch and open (or update) a PR with a body from the `pr` skill.
- **review-changes** — review staged and unstaged changes before committing.
- **take-over-pr** — rebase an agent-opened PR on main, resolve conflicts, quick review.

### Explain

From Taha K's article [Quatre façons d'expliquer un harnais](https://tahak.me/posts/quatre-facons-expliquer-un-harnais/).

- **diagram** — explain a technical subject as a single SVG diagram.
- **sim** — interactive step-by-step HTML page with scenarios.
- **ste** — dense explanation in Simplified Technical English style.
