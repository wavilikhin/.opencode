---
description: Perform comprehensive code review against project standards
---

Execute a comprehensive code review using the @code-reviewer agent, analyzing code against project-specific standards defined in AGENTS.md and any referenced design documents.

## Instructions

### 1. **Determine repository root:**

   - Run `git rev-parse --show-toplevel` to find repository root
   - If not in a git repository, use current working directory as fallback
   - Store this as `$REPO_ROOT`

### 2. **Load project guidelines:**

   - Check for `$REPO_ROOT/AGENTS.md` file
   - If found, read the entire file
   - If not found, warn user and proceed with general best practices only

### 3. **Parse referenced documents:**

   - Scan AGENTS.md for references to other files:
     - Look for patterns like: `See docs/DESIGN.md`, `Refer to guides/API.md`, `Follow rules in X.md`
     - Extract file paths from these references
   - For each referenced file:
     - Check if the reference is relevant to current session/code being reviewed
     - If relevant, attempt to read the file from `$REPO_ROOT/referenced-path`
     - If file exists, include its content in review context
     - If file doesn't exist, note the missing reference but continue

### 4. **Determine scope:**

   - If user specifies files/paths in $ARGUMENTS: review ONLY those files
   - If user provides context (pasted code, specific function): review that context
   - If no specific scope: run `git status` and `git diff` to review current changes
   - Default scope: all uncommitted changes in working directory

### 5. **Invoke code-reviewer agent:**

   - Use Task tool with `subagent_type: "code-reviewer"`
   - Provide comprehensive prompt including:
     - The code/files to review
     - Full content of AGENTS.md
     - Full content of all relevant referenced documents
     - Current session context (what was changed and why)
     - Explicit instruction to check rule relevance before applying
   
   Example prompt structure:
   ```
   Review the following code against project guidelines.
   
   ## Project Guidelines (AGENTS.md):
   [full AGENTS.md content]
   
   ## Referenced Rules:
   [content of each relevant referenced file]
   
   ## Code to Review:
   [code/files being reviewed]
   
   ## Review Instructions:
   - Check each rule's relevance to this specific code
   - Only flag violations of rules that apply to the code being reviewed
   - Distinguish between blocking issues and suggestions
   - Provide specific, actionable fixes with code examples
   
   $ARGUMENTS
   ```

### 6. **Present results:**

   - Display the code-reviewer agent's findings
   - If critical issues found, highlight them clearly
   - If code passes review, confirm compliance

## Examples

### Review specific files:
```
/code-review src/auth/login.ts src/auth/register.ts
```

### Review uncommitted changes:
```
/code-review
```

### Review pasted code:
```
/code-review
[User pastes code in chat, then runs command]
```

### Review with custom focus:
```
/code-review --focus=security,performance
```

## User Instructions

$ARGUMENTS
