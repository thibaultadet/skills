# Numbers behind the choices

Verified 2026-09-24 (platform.claude.com `models/overview`, `optimizing-for-cost-and-intelligence`; code.claude.com `sub-agents`, `model-config`, `workflows`). Re-check when a model ships.

| alias | $/MTok in / out | SWE-bench Pro subset: solved, $ per solved task |
|---|---|---|
| `fable` | 10 / 50 | `low` 88.6 % $0.54 · `high` 92.3 % $1.19 |
| `opus` | 4 / 20 | `low` 87.4 % $0.12 · `medium` 92.8 % $0.22 · `low` then failures at `high`: ~97 % $0.17 (all `high`: 95.3 % $0.29) |
| `sonnet` | 2 / 10 | `high` 77.4 % $0.84 |
| `haiku` | 1 / 5 | no effort, 200K ctx — "high-volume work with checkable outputs, not long agentic loops" |

- Effort is calibrated per model: `low` on one ≠ `low` on another.
- Opus `xhigh` ≈ +1.4 pts over `high` for 2.5× the cost.
- `max` "may show diminishing returns and is prone to overthinking" — not a rung.
- Sonnet: `opus/low` beats it on score at ~⅐ of the cost per solved task — except for writing replay scripts, where it is the default.
- Fable: Opus matched it at ⅕ of the cost. The docs' only case for it — Opus at higher effort still falls short — is a plateau; plateaus go back to the user.

## Model resolution order

Per-call `model` → agent definition's `model` → `CLAUDE_CODE_SUBAGENT_MODEL` → session model. Same for Agent-tool subagents and workflow agents. Effort: workflow `agent()` option → agent definition → session effort.
