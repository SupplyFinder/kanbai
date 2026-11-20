# Notes

Written by `/kanbai setup`; edit freely. Format: `.claude/skills/kanbai/company-files.md`. Example below.

## Data sources (read-only)
- **Postgres** (connector): the app database. Use it to check real table and column names.
Keep queries small (filters, LIMIT). Never copy customer data into code, PRs, tickets, the board or Slack.

## API conventions
- REST under `/api`, JSON. Mirror the sibling resource's auth and validation. Prefer additive changes.

## UI text
- `web`: every new user-facing string in en and fr.

## Notable changes (always listed in a plan, approved with it)
New dependency; breaking API change; database migration; new environment variable; CI or deploy config.

## Environments
- Test environment: `https://dev.example.com` (what is merged into `dev`).
