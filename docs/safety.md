# Safety and Boundaries

## Destructive Actions

Ask before performing:
- Destructive actions (force pushes, hard resets, etc.)
- Wide refactors affecting multiple systems
- Deleting files
- Changing public APIs
- Changing configurations or environment settings
- Adding new dependencies
- Network access or external API calls

## Secrets and Credentials

- Do not print or commit credentials/tokens
- Do not print or commit sensitive environment variables
- Flag suspicious files (e.g., `.env`, `credentials.json`, `.aws/credentials`)
- Warn the user before suggesting commits that might contain secrets

## Never Without Asking

- Never commit/push without explicit user request
- Never commit files that likely contain secrets, even if the user asks (warn them first)
- Never skip git hooks (--no-verify, --no-gpg-sign, etc.) unless the user explicitly requests it
- Never run force push to main/master unless the user explicitly requests it
