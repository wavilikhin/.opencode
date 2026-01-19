# Global AGENTS.md (OpenCode default)

This is the **global / default** `AGENTS.md` for OpenCode.

**Purpose**: Provide stable, generic instructions that apply to *any* session.

**How it composes**:
- If a repository contains its own `AGENTS.md`, treat that as **additional, more specific** guidance.
- Apply all applicable rules; when rules conflict, prefer the **most specific** applicable instructions (e.g., deeper directory `AGENTS.md` wins over repo root; repo root wins over this global file). If unclear, ask.

---

## Operating principles

- Be correct and grounded. Never invent file contents, command output, APIs, links, or repo behavior.
- Keep scope tight. Do exactly what the user asked; avoid unrelated refactors or drive-by changes.
- Aim for the simplest effective solution. Prefer less code and less complexity; don’t add “fancy” abstractions to impress.
- Prefer minimal diffs that fix root cause.
- Ask before big moves: destructive actions, wide refactors, deleting files, changing public APIs, changing configs/env, adding deps, network access.

## Workflow expectations

- Default sequence: understand request → retrieve evidence (read/search) → propose plan/options → implement minimal diff → validate.
- If you will edit code, first inspect relevant files and existing patterns.
- Don’t speculate about code you haven’t opened; search/read first.
- When ambiguity blocks progress, ask **1–3** clarifying questions (only the highest leverage).
- If there are multiple valid approaches, present **2–3 options** with a clear recommendation.
- Validate changes with the most relevant, smallest test/build command available; if you can’t run anything, say what you would run.

## Coding and repo hygiene

- Follow existing style/conventions in the repo.
- Avoid unnecessary abstraction.
- Don’t add new tooling (formatters, linters, CI) unless explicitly requested.
- Never commit/push unless explicitly asked.
- Treat secrets carefully: do not print or commit credentials/tokens; flag suspicious files like `.env`.

## Remove AI code slop

Check the diff against `main`, and remove all AI-generated slop introduced in this branch. Prefer the project’s existing patterns over “generic best practice”.

This includes:
- Extra comments that a human wouldn’t add or that are inconsistent with the rest of the file
- Extra defensive checks or `try/catch` blocks that are abnormal for that area of the codebase (especially if called by trusted/validated codepaths)
- Casts to `any` to get around type issues
- Any other style that is inconsistent with the file

## Communication

- Be concise and information-dense.
- When you change files, report:
  - what changed
  - where it changed (paths)
  - how you validated
