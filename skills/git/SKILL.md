---
name: git
description: Enforce consistent git commit/push/PR workflows. Auto-invoke when the user asks to commit, push, or open a PR without explicitly using slash commands.
---

# Git Workflow Enforcer

## Invocation Policy (Auto)

Enable this skill automatically when any of these are true:
- User asks to **commit** changes ("commit this", "make a commit", "git commit", "as a last step commit")
- User asks to **push** changes ("push this", "git push", "push to origin")
- User asks to **commit and push**
- User asks to **open/create a PR** ("create a PR", "open a pull request")

Do not enable this skill for general git questions (e.g. "what does rebase do?") unless the user is requesting an actual commit/push/PR action.

## Purpose

Ensure the agent follows the SAME standards and safety rules as the repo's existing slash commands:
- `command/commit.md`
- `command/commit-push.md`
- `command/commit-push-pr.md`

This guarantees consistent commit style whether the user invokes `/commit` manually or requests committing implicitly in natural language.

## Canonical Rules (Must Follow)

When committing/pushing/creating PRs, follow the rules from the command templates exactly:

### Commit
Follow `command/commit.md` behavior:

1. **Determine commit style** (precedence order):
   - User-specified format/style in the request is mandatory
   - Else read `COMMIT.md` or `.github/COMMIT.md` if present
   - Else infer from `git log --oneline -10 --no-merges`
   - Else default to Conventional Commits

2. **Decide what to stage**:
   - If user specifies files/patterns, stage ONLY those
   - Else stage ALL changes (`git add -A`)

3. **Analyze changes**:
   - Use `git status`
   - Use diffs (`git diff` / `git diff --staged`) to understand purpose and scope

4. **Match language**:
   - Use `git log --oneline -5` to match language

5. **Commit message composition**:
   - Conventional Commit rules when applicable
   - Add a body for non-trivial changes (>3 files, deps, API change, refactor, breaking)

6. **Pre-commit hooks**:
   - If hooks modify files, stage and retry once

### Commit + Push
If the user requests commit+push, follow `command/commit-push.md`:
- Perform the full commit procedure above
- Then `git push`
- If the remote branch doesn't exist or there are any issues, inform the user

### Commit + Push + PR
If the user requests commit+push+PR, follow `command/commit-push-pr.md`:
- Do the full commit procedure
- Determine current branch (`git branch --show-current`)
- Check remote branch exists (`git ls-remote --heads origin <branch>`)
- Push with `-u` when needed
- Create PR using `gh pr create`
- If `gh` is missing, tell user

## Safety Protocol (Must Follow)

- NEVER change global git config
- NEVER run destructive/irreversible git commands unless explicitly requested (force push, hard reset, rebase -i, etc.)
- NEVER push to `main`/`master` with force; warn and ask confirmation
- NEVER skip hooks (`--no-verify`) unless user explicitly requests
- NEVER commit secrets: warn if `.env`, credentials, tokens, etc. are being staged
- Prefer non-interactive git commands only (no `-i`)

## Execution Behavior

When the user implicitly asks to commit/push/PR:

1. Identify which workflow is requested:
   - commit only  → follow `command/commit.md`
   - commit+push  → follow `command/commit-push.md`
   - commit+push+PR → follow `command/commit-push-pr.md`

2. Execute the same git discovery commands the slash commands would:
   - `git status`
   - `git diff --staged` and `git diff`
   - `git log --oneline -5`
   - (For PR) `git branch --show-current`, `git ls-remote --heads ...`

3. Draft the commit message that matches project conventions.

4. Only create commits/push/PR when the user clearly requested it.

## Output Requirements

After completing a git workflow, report:
- What was committed (high level)
- The commit hash
- Whether push occurred and which branch/remote
- PR URL (if created)
- Any warnings (secrets detected, missing upstream, hooks changed files)
