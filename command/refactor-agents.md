---
description: Refactor AGENTS.md to follow progressive disclosure principles, then commit and push
---

Refactor an AGENTS.md file to follow progressive disclosure principles by identifying contradictions, extracting essentials, grouping related instructions, and creating a modular documentation structure. Then commits and pushes the changes to the remote repository.

## Instructions

### Refactoring Phase

#### 1. **Locate AGENTS.md file:**

   - If user specifies a path in $ARGUMENTS: use that path
   - Otherwise, check for `$REPO_ROOT/AGENTS.md`
   - If not found, check for `AGENTS.md` in current directory
   - If still not found, ask user for the file path

#### 2. **Find contradictions:**

   - Read the entire AGENTS.md file
   - Identify any instructions that conflict with each other
   - For each contradiction found:
     - Document both conflicting statements
     - Present them to the user
     - Ask which version they want to keep
     - Record their response

#### 3. **Identify essentials:**

   - Extract instructions that belong in root AGENTS.md:
     - One-sentence project description (if present)
     - Package manager (if not npm)
     - Non-standard build/typecheck commands
     - Core operating principles that apply to every task
     - Communication style guidelines
   - Flag anything else as candidate for grouping

#### 4. **Group related instructions:**

   - Organize remaining instructions into logical categories such as:
     - Workflow expectations
     - Code quality and style
     - Testing patterns
     - API design
     - Git workflow
     - Safety and boundaries
     - Type system conventions
   - Present groupings to user for validation
   - Allow user to suggest alternative groupings

#### 5. **Create separate files:**

   - Create `docs/` folder if it doesn't exist
   - For each group, create a markdown file with:
     - Clear section headings
     - Organized, scannable content
     - Cross-references to related files where appropriate
   - File naming convention: `docs/{category}.md`

#### 6. **Flag for deletion:**

   - Identify instructions that are:
     - Redundant (already covered by other principles)
     - Too vague to be actionable
     - Overly obvious (e.g., "write clean code")
   - Present deletions to user with reasoning
   - Ask for confirmation before removing

#### 7. **Create refactored root AGENTS.md:**

   - Reduce root file to essentials + links pattern
   - Add markdown links to new documentation files
   - Maintain composition rules (if present)
   - Preserve core principles
   - Keep communication guidelines concise

#### 8. **Generate output summary:**

   - Display folder structure created
   - List files created/modified
   - Show before/after line counts for root AGENTS.md
   - Provide summary of changes per category

#### 9. **Ask final validation:**

   - Are you satisfied with the groupings?
   - Should any instructions be moved between files?
   - Do you want to make additional adjustments?
   - Ready to commit these changes?

### Commit Phase

#### 10. **Execute commit process:**

   Follow all instructions from `/commit` command to create a commit that documents the refactoring:

   **Commit message guidance:**
   - Type: `docs` (for documentation refactoring)
   - Scope: `agents` or `config` (depends on repo structure)
   - Description: Summarize the refactoring (e.g., "refactor AGENTS.md with progressive disclosure structure")
   - Body: Explain the changes:
     - Number of new documentation files created
     - Root AGENTS.md reduction (line count before/after)
     - Categories extracted to separate files
     - Why: improves maintainability and follows progressive disclosure pattern

   Example commit:
   ```
   docs(agents): refactor AGENTS.md with progressive disclosure structure

   Extract detailed instructions into modular docs/:
   - docs/workflow.md: evidence-gathering and validation patterns
   - docs/code-quality.md: AI slop removal and style conventions
   - docs/safety.md: boundaries and destructive action safeguards

   Root AGENTS.md reduced from 55→38 lines. All functionality preserved via cross-links.
   ```

### Push Phase

#### 11. **After successful commit:**

   Push the changes to the current branch:

   ```bash
   git push
   ```

   If the remote branch doesn't exist, use:
   ```bash
   git push -u origin <branch-name>
   ```

   If push fails or branch tracking is missing, inform the user with the error details.

#### 12. **Completion confirmation:**

   - Confirm successful push to remote
   - Display branch name and commit hash
   - Provide instructions for next steps (e.g., "Ready to create a PR with `/commit-push-pr`")

## Examples

### Refactor AGENTS.md in current repo and commit+push:
```
/refactor-agents
```

### Refactor specific AGENTS.md file:
```
/refactor-agents /path/to/AGENTS.md
```

### Refactor with custom grouping preferences:
```
/refactor-agents --preferences="workflow,safety,code-quality,testing"
```

## Decision Points

The command will ask for input at these moments:
1. **Contradiction resolution**: Which version of conflicting instructions to keep
2. **Grouping validation**: Are the proposed categories correct?
3. **Deletion confirmation**: Should redundant/vague instructions be removed?
4. **File structure approval**: Is the output structure acceptable?
5. **Commit readiness**: Are you ready to commit these changes?

## Output Structure

After successful refactoring and push, you will have:

```
AGENTS.md (refactored - reduced to essentials with links)
docs/
├── workflow.md (evidence-gathering, ambiguity handling, validation)
├── code-quality.md (patterns, slop removal, style)
├── safety.md (boundaries, destructive actions, secrets)
└── [additional category files as needed]
```

All changes will be committed and pushed to the remote branch.

## User Input

$ARGUMENTS
