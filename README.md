# OpenCode Config

Personal OpenCode configuration repository (synced via git).

## Layout

- `opencode.json` / `configs/`: OpenCode configuration (schema: https://opencode.ai/config.json)
- `commands/`: Custom slash commands (`.md` files)
- `command/`: Prompt templates used by workflows (commit flows, review flow)
- `skills/`: Agent skills (auto/manual behaviors)
- `agent/`: Agent definitions (primary/subagents)
- `plugin/`: Local plugins (if present)

## Slash Commands

Slash commands are invoked explicitly in the TUI by typing `/`.
Docs: https://opencode.ai/docs/commands/

- `/bug-fix` (`commands/bug-fix.md`)
  - Purpose: structured bug-fixing workflow (root cause first, minimal diff, suggest tests).
  - Invocation: manual (`/bug-fix <context>`).

- `/code-review` (`command/code-review.md`)
  - Purpose: run a comprehensive review via the `code-reviewer` subagent.
  - Invocation: manual (`/code-review [paths or notes]`).

## Workflow Templates (command/)

These are prompt templates referenced by other workflows (and by the `git` skill).

- `/commit` template: `command/commit.md`
- `/commit-push` template: `command/commit-push.md`
- `/commit-push-pr` template: `command/commit-push-pr.md`

## Agents

Agent files live in `agent/`.
Docs: https://opencode.ai/docs/agents/

- `ask` (`agent/ask.md`)  primary, read-only Q&A agent.
- `code-reviewer` (`agent/code-reviewer.md`)  subagent invoked only via `/code-review`.
- `system-architect` (`agent/system-architect.md`)  subagent for high-level architecture/design.

## Skills

Skills are behavior modules that can be auto-invoked based on context.
Docs: https://opencode.ai/docs/skills/

- `git` (`skills/git/SKILL.md`)
  - Auto-invoke: when the user asks to commit/push/open PR (even without slash commands).
  - Dependency links:
    - Follows the same rules as `command/commit.md`
    - For commit+push: follows `command/commit-push.md`
    - For commit+push+PR: follows `command/commit-push-pr.md`

- `code-simplifier` (`skills/code-simplifier/SKILL.md`)
  - Use: after writing/modifying code; simplifies while preserving behavior.

- `nextjs-guru` (`skills/nextjs-guru/SKILL.md`)
  - Use: Next.js 16+ App Router expert (manual-only).

- `reatom-guru` (`skills/reatom-guru/SKILL.md`)
  - Use: Reatom expert; auto-enable when project uses `reatom` or when working on Reatom code.

- `brainstorming` (`skills/brainstorming/SKILL.md`)
  - Use: manual brainstorming partner.

- `prompt-optimizer` (`skills/prompt-optimizer/SKILL.md`)
  - Use: rewrite prompts into execution-ready agent prompts (manual-only).

- `frontend-designer` (`skills/frontend-designer/SKILL.md`)
  - Use: UI/UX and frontend design support.

## Key Links Between Components

- `command/code-review.md`  invokes `code-reviewer` subagent (`agent/code-reviewer.md`).
- `skills/git/SKILL.md`  enforces commit flow rules from:
  - `command/commit.md`
  - `command/commit-push.md`
  - `command/commit-push-pr.md`

## References

- Config: https://opencode.ai/docs/config/
- Commands: https://opencode.ai/docs/commands/
- Skills: https://opencode.ai/docs/skills/
- Agents: https://opencode.ai/docs/agents/
- Plugins: https://opencode.ai/docs/plugins/
