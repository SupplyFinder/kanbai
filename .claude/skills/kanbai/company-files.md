# Company files (format)

The company repo holds four files in `kanbai/`. The setup command writes them; people may edit them.
They are plain text so they read well on GitHub.

## config.yml
```yaml
company: <name shown on the board and in messages>
owner_name: <first name of the board owner>
board_url: <https://claude.ai/artifact/…>
ticket_prefix: <letters, e.g. ACME>          # tickets are ACME-1, ACME-2…
timezone: <IANA zone, e.g. Europe/Paris>
session_tag: <tag on every pipeline session, e.g. KanbAI>
github_owner: <GitHub organisation or user>
company_repo: <owner/repo of the company repo>
schedules:
  daily: "<minute hour * * *>"               # required: daily reconcile and backup
  hourly: "<minute hours * * days>"          # optional: hourly reconcile in working hours
slack:
  enabled: true | false
  channel: <channel id>                      # token in the SLACK_BOT_TOKEN environment variable
backups:
  repo: <owner/repo>                         # private repo for the daily backup
  branch: daily
screenshots:
  enabled: true | false
  repo: <owner/repo>                         # where PR screenshots are committed
  branch: screenshots
forbidden_repos: [<owner/repo>, …]           # never read, attach or change
dry_run: false                               # true: code is pushed to branches but no PR is opened
```

## repos.md
A `## Coding order` section (keys in order, APIs before the UIs that call them), then one `## <key>`
section per product repo with these lines (omit what does not apply):
- `Repo: owner/name · Label: <shown on the board>`
- `What it is: <one line>`
- `PR target: <branch> · Release branch: <branch> · Branches: <pattern with T-n and <slug>>`
- `Depends on: <key> (attach it read-only when coding this repo)`
- `Checks (mirror CI): <commands>`
- `Baseline: <errors that already exist and do not count>`
- `Layout: <where things go; skills or AGENTS.md to follow>`
- `Rules: <things never to do in this repo>`
An optional `## Hints` section ends the file.

## notes.md
Sections, each optional: `## Data sources (read-only)` (one line per connector: what it holds), `## API
conventions`, `## UI text` (languages per repo), `## Notable changes` (what a plan must list for approval),
`## Environments` (dev URLs), and any other team rules.

## screenshots.md
Which repos get screenshots, how to log in, how to reach a preview or run the app locally (Real mode), how
to stub the backend (Stubbed mode), and what must never appear in a screenshot.
