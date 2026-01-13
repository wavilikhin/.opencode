---
description: Commit, create remote branch if needed, push, and create PR
---

Follows the same process as `/commit`, then ensures the branch exists on the remote, pushes changes, and creates a pull request with a description of the changes.

## Instructions

### Execute commit process:
Follow all instructions from `/commit` command to create the commit.

### After successful commit:

1. **Get current branch name:**
   ```bash
   git branch --show-current
   ```

2. **Check if remote branch exists:**
   ```bash
   git ls-remote --heads origin <branch-name>
   ```

3. **If branch doesn't exist remotely, create and push:**
   ```bash
   git push -u origin <branch-name>
   ```

   **OR** if branch exists, just push:
   ```bash
   git push
   ```

4. **Create pull request using GitHub CLI:**

   Get the commit history for the branch to generate the PR description:
   ```bash
   git log origin/main..<branch-name> --oneline
   ```

   Or if the base branch might be different:
   ```bash
   git rev-parse --abbrev-ref --symbolic-full-name @{u}
   ```

   Create PR using `gh pr create`:
   ```bash
   gh pr create --title "<commit-message-title>" --body "$(cat <<'EOF'
   <PR description based on commits and changes>
   EOF
   )"
   ```

   **PR description should include:**
   - Summary of changes based on commit messages
   - Key features or fixes
   - List of commits (if there are multiple)

   If only one commit exists, reuse the commit message as the PR title and description body.

5. **Error handling:**
   - If `gh` is not installed, inform the user
   - If branch is already up to date, inform the user
   - If PR creation fails, provide the error message

## User Instructions

$ARGUMENTS