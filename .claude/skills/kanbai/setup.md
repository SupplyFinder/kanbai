# Setup and repo commands

Run by a person in a Claude Code session started on the company repo (a copy of the KanbAI template).
Commands: `/kanbai setup`, `/kanbai add-repo <owner/name>`, `/kanbai remove-repo <key>`,
`/kanbai refresh-repos`. They only ever change files of the company repo (through a branch and a PR)
and the board's settings documents; never a product repo. Ask with AskUserQuestion when available,
several questions at once, each pre-filled with what you detected.

## /kanbai setup

### 1. Check where you are
- The session must be on the company repo (it holds `.claude/skills/kanbai/`). If
  `kanbai/config.yml` is already filled in, say so and offer `refresh-repos` instead; continue only if
  the person confirms a full re-setup.
- GitHub is required: list the repos this session can reach (`mcp__claude-code-remote__list_repos`). None →
  stop and explain: connect GitHub in claude.ai and install the Claude GitHub App on the repos.

### 2. Basics (one round of questions)
Company name; the board owner's first name (the person running setup, from their profile); ticket prefix
(suggest 2–4 capitals from the company name); time zone (suggest from the person's locale); Slack
notifications yes/no (then the channel id, and the bot token goes in the environment variable
`SLACK_BOT_TOKEN`, never in chat); PR screenshots yes/no; the private backup repo name (suggest
`<company repo>-backup`; it must exist, the person creates it on GitHub).

### 3. Pick the repos
Show the reachable repos (name, last push) and ask which ones the pipeline may work on. Never offer the
company repo itself or the backup repo.

### 4. Read each picked repo (read-only)
Attach it read-only (`add_repo`, clone, one at a time) and work out, writing down where each fact came from:
- **What it is**: framework from `package.json`, `pyproject.toml`, `go.mod`, …, README and folders.
- **Branches**: the default branch; the base branch of the last ~30 merged PRs (`list_pull_requests`,
  state closed); branches CI deploys from (`.github/workflows`: "on push to X → deploy"); the naming of
  recent PR branches (`feature/…`, `feat/…`, `<KEY>-123-…`). Propose: PR target = where most PRs go;
  release = the branch production deploys from (often the default branch); branch pattern =
  `<their prefix style>/T-n-<slug>`.
- **Checks**: the commands CI runs on PRs (workflow steps); else `package.json` scripts, `Makefile`,
  `pre-commit`, `pyproject` tool sections. Do not run them now (it is slow); the baseline line stays empty
  unless the person asks for a baseline run.
- **Dependencies and coding order**: a UI's API base URL setting (e.g. `VITE_API_URL`, `NEXT_PUBLIC_API_URL`)
  or API client pointing at another picked repo; shared types. APIs come before the UIs that call them.
- **Layout**: `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING`, repo skills; the folders and the files that change
  together in the last ~20 merged feature PRs ("new endpoint = model + service + route + router").
- **UI text**: translation folders and their languages.
- **Outside services**: dependencies (`@supabase/supabase-js`, `pg`, `psycopg`, Prisma `provider`,
  `clickhouse-connect`, `pymongo`/`mongoose`, `@sentry/*`, `stripe`…), env names in `.env.example`
  (`SUPABASE_URL`, `DATABASE_URL`, `CLICKHOUSE_HOST`, `MONGODB_URI`, `SENTRY_DSN`…), `docker-compose`
  services, `supabase/` or `prisma/` folders.
- **Branch protection**: from `list_branches` (`protected`) for the PR target and release branches.

### 5. Confirm branches (one question)
"Claude's PRs go into: `<detected>` · Production branch: `<detected>`" for all repos, then "Any repo
different?" listing only the repos whose detection differs or is unclear (no PR history). For a repo whose
PR target IS its release branch (main-only teams): warn that merging a Claude PR releases it, keep PRs as
drafts, and recommend branch protection with required reviews.

### 6. Connectors
Compare the outside services found with the connectors available in this session (and, if a tool exists
to list the account's connectors, those):
- **Required**: GitHub.
- **Recommended**: each data store the code uses (Supabase, Postgres, ClickHouse, MongoDB, BigQuery,
  Snowflake…), read-only, so planning can check real names and shapes.
- **Nice to have**: error tracking (Sentry…) for bug tickets; Slack if enabled; an issue tracker for imports;
  product analytics.
- **Not needed**: payment, email and other services the app calls.
Say for each: connected, available to add (claude.ai → Settings → Connectors; the person adds it, never
you), or no connector (planning works from the code). Never suggest putting database passwords in the
environment.

### 7. Logo and colour
Candidates, in order: the GitHub organisation's avatar; the website (organisation `blog` field, or the
domain of the person's email): its header logo (SVG/PNG), `apple-touch-icon`, favicon, social image. Show
the candidates (save each to a file and show it) and ask: use one, upload my own later (Settings →
Branding), or none. Suggest the main colour from the chosen logo. For dark mode, if the logo has dark text,
make a light-text copy. A website the network blocks: say so and fall back to the avatar or none.

### 8. Write the company files
Draft `kanbai/config.yml`, `repos.md`, `notes.md` (data sources found, with "not connected" where so; UI
text; notable changes; environments; then a few questions the code cannot answer: what Claude must never
change without asking, who reviews PRs) and `screenshots.md` (only if enabled: which repos, how to log in,
preview URLs if PR checks show preview deployments). Format: `company-files.md`. Show the drafts, marking
each line "detected" or "you said", and apply the person's edits.

### 9. Permission rules
Write `.claude/settings.json` from `settings.base.json` (next to this file; drop its `_note`) plus, for this company:
- allow: each repo's check commands (as `Bash(<command prefix>:*)`), the install commands they need, and
  the data connectors' tools (`mcp__<connector>`);
- deny, for every repo: `Bash(git push origin <PR target>:*)`, `Bash(git push origin <release>:*)`, the same
  with `-u` / `--set-upstream`, force pushes and branch deletions;
- never allow merging PRs or turning on auto-merge.
These rules catch plain `git push` lines only; the real protection is GitHub branch protection (step 4):
list unprotected PR targets and release branches and recommend protecting them.

### 10. Publish the board
Publish `board/board.html` as a new artifact (`Artifact` publish, `icon: "kanban"`, a one-line
description) with the capabilities in `board/rules.json`. Put its URL in `config.yml` `board_url`. Then
seed its settings with `ArtifactData`: `config/company` (orchestrator.md, step 1), `config/models` (the
defaults in board.md), `config/runtime` (`slack_channel` when Slack is on), `config/branding` (upload the
chosen logo files to the board's assets first, then `{logo_light, logo_dark, accent}`).

### 11. Hand over
Commit the company files and `.claude/settings.json` on a new branch, push it, and give the person the
link to open a PR (open it yourself only if they ask). Then list what is left for them:
1. merge the PR;
2. share the board with the organisation (the board's Share menu);
3. start the orchestrator in a NEW session on the company repo: `/kanbai orchestrator <board url>`;
4. teammates open the board once; tick approvers and admins in Permissions;
5. optional: upload the skill as an organisation skill for "create a ticket from our discussion" in any
   chat (the README explains the zip);
6. first ticket: something small, with `dry_run: true` in `config.yml` if they want to see the code before
   any PR exists.
Also say once: when the orchestrator restarts it moves its scheduled checks; old ones it cannot delete are
disabled and can be deleted by hand from the scheduled tasks list.

## /kanbai add-repo <owner/name>
Steps 4–6 for that repo only (and the branch question for it), then: add its section to `repos.md` and its
place in "Coding order"; new data sources to `notes.md`; its check commands and branch denies to
`.claude/settings.json`; update `config/company.repos` on the board. Commit on a branch, push, give the PR
link. No orchestrator restart is needed: sessions read the company files when they start.

## /kanbai remove-repo <key>
Remove its section and its place in "Coding order", its branch denies and check commands (keep commands
other repos use), and update `config/company.repos`. Tickets in progress that touch it keep their plan; say
which ones. Open PRs in that repo are left alone. Branch, push, PR link.

## /kanbai refresh-repos
Steps 4–6 for every repo in `repos.md`, then propose only the differences (new checks, moved folders, new
services, changed default branches) as one PR. Never drop a line the person wrote by hand without asking.
