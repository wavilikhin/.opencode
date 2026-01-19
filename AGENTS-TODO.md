# AGENTS.md

This repository is a personal OpenCode configuration repo (tracked in git so it can be synced across machines).

CRITICAL: Keep this `AGENTS.md` file auto-updated on any significant codebase changes.
CRITICAL: Keep `Readme.md` auto-updated on any significant codebase changes.

## What lives where

- `opencode.json` / `configs/`: OpenCode configuration (schema: https://opencode.ai/config.json)
- `commands/`: Custom slash commands (`.md` files). Name = command (e.g. `commands/bug-fix.md` → `/bug-fix`)
- `command/`: Prompt templates used by the workflow (e.g. commit flows)
- `skills/`: Agent skills (auto/manual behaviors)
- `plugin/` / `agent/`: Local extensions and agent definitions (if present)

## Conventions

- Keep prompts short and operational (no long prose).
- Prefer minimal, focused changes; avoid unrelated refactors.
- When adding a new command/skill, keep the scope tight and include clear invocation rules.

## Git workflows (important)

- For commit/push/PR flows, follow the repo templates:
  - `command/commit.md`
  - `command/commit-push.md`
  - `command/commit-push-pr.md`
- If user requests commit/push implicitly (natural language), the `git` skill should apply the same rules.

## Useful docs

- Commands: https://opencode.ai/docs/commands/
- Skills: https://opencode.ai/docs/skills/
- Agents: https://opencode.ai/docs/agents/
- Plugins: https://opencode.ai/docs/plugins/
- Config: https://opencode.ai/docs/config/
