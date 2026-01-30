# Code Quality

## Remove AI Code Slop

Check the diff against `main`, and remove all AI-generated slop introduced in this branch. Prefer the project's existing patterns over "generic best practice".

This includes:
- Extra comments that a human wouldn't add or that are inconsistent with the rest of the file
- Extra defensive checks or `try/catch` blocks that are abnormal for that area of the codebase (especially if called by trusted/validated codepaths)
- Casts to `any` to get around type issues
- Any other style that is inconsistent with the file

**Note**: This was added because earlier models (pre-GPT 5.2 and Anthropic Opus 4.5) tended to add unnecessary comments and patterns that didn't match team voice and style. Modern models are better at this, but the check remains important for maintaining consistency.

## Style and Patterns

When writing or modifying code:
- Follow existing style and conventions in the repository
- Prefer the project's established patterns over generic best practices
- Minimize complexity; keep solutions simple and direct
