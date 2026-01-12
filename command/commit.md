---
description: Create a git commit following project conventions
---

Create a git commit that respects project conventions, inferred style, and commit message language.

## Instructions

### 0. **Determine commit style (precedence order):**

   1. **User instructions:** If user specifies format/style in $ARGUMENTS, follow it strictly
   2. **COMMIT.md:** Check for `COMMIT.md` or `.github/COMMIT.md` in repo root
      - If found, read and follow its rules strictly
      - Treat as mandatory requirements
   3. **Infer from history:** If no COMMIT.md exists:
      - Run `git log --oneline -10 --no-merges` 
      - Analyze last 10 commits for:
        - Format pattern (conventional commits, semantic, gitmoji, free-form)
        - Language (English, Russian, Chinese, etc.)
        - Scope usage patterns
        - Body frequency and style
        - Average description length
      - Match the detected style
   4. **Default:** Use conventional commits if repo has <3 commits or no clear pattern

### 1. **Determine files to commit:**

   - If user specifies files or patterns: stage ONLY those files
   - If user provides no file guidance: stage ALL changed files (`git add -A`)
   - User instructions about file selection are MANDATORY and override defaults

### 2. **Analyze changes:**

   - Run `git status` to see what will be committed
   - Run `git diff --staged` (or `git diff` before staging) to understand changes
   - Identify the primary purpose: feature, fix, refactor, docs, etc.
   - Count files changed and assess scope complexity

### 3. **Detect commit message language:**

   - Run `git log --oneline -5` to check language of recent commits
   - Match language to repo's commit history (e.g., Russian repo → Russian commits)
   - For English repos or no history: use English
   - Maintain language consistency with existing commits

### 4. **Compose commit message:**

   **If following Conventional Commits (default):**

   ```
   <type>[optional scope]: <description>

   [optional body]
   ```

   **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

   **Scopes:** Infer from codebase structure or use: `auth`, `storage`, `ui`, `spaces`, `groups`, `bookmarks`, `extension`

   **Rules:**
   - Lowercase type and scope
   - Imperative mood ("add" not "added")
   - No period at end of description
   - Description max ~50 chars
   - Use detected language consistently

   **If using inferred style:** Match format, structure, and language exactly

### 5. **Add body for non-trivial changes:**

   **ALWAYS add body when:**
   - Changes span >3 files
   - Adding/removing dependencies
   - Changing APIs, interfaces, or data structures
   - Refactoring architecture or core logic
   - Any breaking changes
   - Feature additions or significant fixes

   **Body guidelines:**
   - Explain WHAT changed and WHY (not how)
   - Max 2-3 sentences, information-dense
   - Provide context for future developers and AI agents
   - Separate from description with blank line
   - Match language of commit description

   **SKIP body only for:**
   - Typo fixes (single word/character changes)
   - Formatting/linting auto-fixes
   - Version bumps with no logic changes
   - Simple one-line fixes

### 6. **Breaking changes:**

   - Add `!` after type/scope: `feat(storage)!: migrate to IndexedDB`
   - Or add footer: `BREAKING CHANGE: description`

### 7. **Execute commit:**

   ```bash
   git add <files>  # or git add -A
   git commit -m "<type>(scope): description" -m "Body explaining what and why."
   ```

### 8. **If pre-commit hooks modify files:** 

   - Retry commit once to include those changes
   - Run `git add -A && git commit -m "..." -m "..."`

## Examples

### English repo (conventional commits):

Simple fix:
```bash
git add -A
git commit -m "fix(bookmarks): prevent duplicate entries on rapid clicks"
```

Feature with body:
```bash
git add src/components/new-tab/
git commit -m "feat(spaces): add color customization" -m "Allows users to assign colors to spaces for visual organization. Colors persist in IndexedDB and sync across sessions."
```

Refactor:
```bash
git add -A
git commit -m "refactor(hooks): extract shared selector logic into base hook" -m "Reduces duplication across use-spaces, use-groups, use-bookmarks by centralizing filter and sort operations."
```

### Russian repo example:

```bash
git add src/auth/
git commit -m "feat(auth): добавить OAuth аутентификацию" -m "Реализована поддержка OAuth через Google и GitHub. Токены сохраняются в зашифрованном виде в локальном хранилище."
```

### Repo with custom COMMIT.md:

If COMMIT.md specifies emoji + imperative format:
```bash
git commit -m "✨ Add dark mode toggle" -m "Users can now switch between light and dark themes in settings."
```

## User Instructions

$ARGUMENTS
