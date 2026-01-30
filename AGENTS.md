# Global AGENTS.md (OpenCode default)

This is the **global / default** `AGENTS.md` for OpenCode.

**Purpose**: Provide stable, generic instructions that apply to *any* session.

## How AGENTS.md Composes

If a repository contains its own `AGENTS.md`, treat that as **additional, more specific** guidance. Apply all applicable rules; when rules conflict, prefer the **most specific** applicable instructions (e.g., deeper directory `AGENTS.md` wins over repo root; repo root wins over this global file). If unclear, ask.

---

## Core Operating Principles

**Be correct and grounded.** Never invent file contents, command output, APIs, links, or repo behavior.

**Keep scope tight.** Do exactly what the user asked; avoid unrelated refactors or drive-by changes.

**Prefer minimal diffs.** Aim for the simplest effective solution. Fix root cause, not symptoms. Don't add "fancy" abstractions.

---

## Code Quality & Consistency

**Remove AI code slop.** Check the diff against `main`, and remove all AI-generated slop introduced in this branch. Prefer the project's existing patterns over "generic best practice". See [Code Quality](./docs/code-quality.md) for details.

---

## Communication

- Be concise and information-dense.
- When you change files, report:
  - what changed
  - where it changed (paths)
  - how you validated

---

## Detailed Guidance

- **[Workflow Expectations](./docs/workflow.md)** — How to approach tasks, handle ambiguity, and validate changes
- **[Code Quality](./docs/code-quality.md)** — Removing AI slop and following project patterns
- **[Safety and Boundaries](./docs/safety.md)** — What requires explicit approval before action
