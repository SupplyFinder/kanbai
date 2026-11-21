# Setup and repo commands

Run by a person in a Claude Code session started on the company repo (a copy of the KanbAI template).
Commands: `/kanbai setup`, `/kanbai add-repo <owner/name>`, `/kanbai remove-repo <key>`,
`/kanbai refresh-repos`. They only ever change files of the company repo (through a branch and a PR)
and the board's settings documents; never a product repo.

**Ask as little as possible.** Ask only what is mandatory and cannot be detected or defaulted: which repos
the pipeline may work on (step 3), and who gets the chat skill (step 8). Ask about branches
(step 5) only when detection is unclear. Never ask the person to review or confirm the drafts, and never
ask for a Slack channel. Everything else takes the default below, is shown in the step 8
summary marked "default", and can be changed later in `kanbai/config.yml` or in the setup PR (the orchestrator picks it up
on its next daily check or restart) or, for logo and colour, in the board's Settings → Branding. Ask with
AskUserQuestion when available, all questions of a step in one call, the detected answer first.

## /kanbai setup

### 1. Check where you are
- The session must be on the company repo (it holds `.claude/skills/kanbai/`). If
  `kanbai/config.yml` is already filled in, say so and offer `refresh-repos` instead; continue only if
  the person confirms a full re-setup.
- GitHub is required: list the repos this session can reach (`mcp__claude-code-remote__list_repos`). None →
  stop and explain: connect GitHub in claude.ai and install the Claude GitHub App on the repos.

### 2. Basics (no questions: defaults)
- Company name: the GitHub organisation's display name (else its login, readable).
- Owner: the first name of the person running setup (their profile).
- Ticket prefix: `KB` (tickets KB-1, KB-2…). Only if the picked repos already have branches or PRs named
  `…/KB-<number>-…` (another pipeline), use `KB2`, `KB3`… Changing it later in `config.yml` only affects new
  tickets.
- Time zone: from the person's locale or email domain country; else `UTC`.
- Slack: off (no question). If `SLACK_BOT_TOKEN` is set in this environment, say in the summary that Slack can be
  turned on by setting `slack.enabled: true` and `slack.channel` in `config.yml`.
- PR screenshots: on, `screenshots.repo` = the company repo, branch `screenshots`. Screenshots are only
  taken for tickets whose author ticks "Add screenshots to the PRs", and only in repos `screenshots.md`
  lists (the UI repos found in step 4). With no UI repo picked, leave it off.
- Backups: board only (30 days); `backups.repo` empty. Say in the summary that a private GitHub repo can be
  added later for a permanent copy.

### 3. Pick the repos (the one mandatory question)
Show the reachable repos (name, last push) and ask which ones the pipeline may work on (multi-select;
pre-select the ones pushed in the last 30 days). Never offer the
company repo itself or the backup repo (if any).

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

### 5. Branches (ask only when unclear)
No question when every repo's PR target and release branch are clear from its PR history and CI: show
them in the step 8 summary. Ask only about the repos whose detection is unclear (no PR history, or PRs
split between branches), all in one question. For a repo whose
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
the candidates (save each to a file and show it). No question: use the first good candidate (a wordmark
over a square icon), say which in the summary, and that it can be changed in the board's Settings → Branding. Suggest the main colour from the chosen logo. For dark mode, if the logo has dark text,
make a light-text copy. A website the network blocks: say so and fall back to the avatar or none.

### 8. Write the company files
Draft `kanbai/config.yml`, `repos.md`, `notes.md` (data sources found, with "not connected" where so; UI
text; notable changes; environments; then a few questions the code cannot answer: what Claude must never
change without asking, who reviews PRs) and `screenshots.md` (when screenshots are on: which repos, how to log in,
preview URLs if PR checks show preview deployments). Format: `company-files.md`. For the questions the code
cannot answer, write a default ("Notable changes" list; reviews: the owner) instead of asking.
Show the drafts in one message, marking each line "detected", "default" or "you said", as a summary, not
a question: no "Looks right?" confirmation and no Slack channel question. Ask ONE question only, the chat
skill (step 10a), then go on. The person changes anything else later in the company files (or in the setup PR).

### 9. Permission rules
Write `.claude/settings.json` from `settings.base.json` (next to this file; drop its `_note`) plus, for this company:
- allow: each repo's check commands (as `Bash(<command prefix>:*)`), the install commands they need, and
  the data connectors' tools (`mcp__<connector>`);
- deny, for every repo: `Bash(git push origin <PR target>:*)`, `Bash(git push origin <release>:*)`, the same
  with `-u` / `--set-upstream`, force pushes and branch deletions;
- never allow merging PRs or turning on auto-merge.
These rules catch plain `git push` lines only; the real protection is GitHub branch protection (step 4):
list unprotected PR targets and release branches in the step 8 summary (no question): the board
shows a warning until they are protected on GitHub. If the person says later that protection is
not wanted, add `Protection: not required for <branch>` to that repo's section in `repos.md`; the board
then shows a note instead.

### 10. Publish the board
Publish `board/board.html` as a new artifact (`Artifact` publish, `icon: "kanban"`, a one-line
description) with the capabilities in `board/rules.json`. Put its URL in `config.yml` `board_url`. Then
seed its settings with `ArtifactData`: `config/company` (orchestrator.md, step 1), `config/models` (the
defaults in board.md), `config/runtime` (`slack_channel` when Slack is on), `config/branding` (upload the
chosen logo files to the board's assets first, then `{logo_light, logo_dark, accent}`).
Publishing made this setup session watch the board: stop that watch right away (`ArtifactComments`
`watch` with the board URL and `on: false`), so the board's comments wake only the orchestrator.

### 10a. Installed skill (the one question asked in step 8)
The pipeline needs the skill installed on the owner's account: coding sessions run on the product repos, not
the company repo, and load `/kanbai` and the company files from that installed copy. It also gives everyone it
is shared with "create a ticket in the kanban from our discussion" in any Claude chat. Ask who gets it:
"The whole organisation (Recommended)" or "Only me" (the owner at least; no "Not now"). First build a plugin
folder in the scratchpad (never in the repo): `kanbai-chat/.claude-plugin/plugin.json` = `{"name": "kanbai",
"version": "<skill version>", "description": "KanbAI for the <company> board: ticket sessions and tickets from
any Claude chat."}`, and `kanbai-chat/skills/kanbai/` = every file of this skill folder plus the four company
files `kanbai/config.yml`, `repos.md`, `notes.md`, `screenshots.md` copied next to `SKILL.md` (the skill reads
them there outside the company repo). No secret.
- **Organisation**, when the `PublishPlugin` tool is available (load it with ToolSearch if it is only
  listed by name): `PublishPlugin` with `path` = that folder, `destination: "organization"`. The person
  approves the upload in the tool's own dialog; pass its result lines on as they are (an admin may have to
  approve it in the organisation library). If it says a `kanbai` plugin already exists, ask before
  `replace: true`. Without the tool, or if it is refused: do as for "Only me" and say an org admin can
  upload the same zip as an organisation skill.
- **Only me**: zip `skills/kanbai/` from that folder, send it to the person (`SendUserFile`), and say to
  upload it in claude.ai as a personal skill (Settings → Skills). `PublishPlugin` only publishes to the
  organisation, so never use it for this choice.
In chats only the ticket role works (chat-ticket.md); whoever uses it needs access to the board.

### 11. Hand over
Commit the company files and `.claude/settings.json` on a new branch, push it, and give the person the
link to open a PR (open it yourself only if they ask), plus the board link. Then end the message with these
three sections, in this order and with these headings:

**Connectors**: one line per service from step 6: connected, available to add (claude.ai → Settings →
Connectors), failing to connect, or no connector (planning works from the code).

**Defaults used**: a table with columns Setting · Value · Where to change. Rows: ticket prefix, time zone,
Slack, screenshots, backups, schedules, dry run, logo and colour. Where to change: `kanbai/config.yml`
(logo and colour: the board's Settings → Branding). Under the table, say: "Edit `kanbai/config.yml` directly,
or ask me here and I update it (on the setup branch before the PR is merged, on a new branch and PR after)."

**What's left for you**:
1. merge the PR;
2. share the board with the organisation (the board's Share menu);
3. start the orchestrator in a NEW session on the company repo with this message, filled in (give it
   ready to paste in a code block, one repo per `owner/name`, every repo in `repos.md`, none from
   `forbidden_repos`): `/kanbai orchestrator <board url>` and, on the next line,
   `Attach with push access: <owner/name>, …`;
4. teammates open the board once; tick approvers and admins in Permissions;
5. install the skill from step 10a (approve it in the organisation library if asked, or upload the zip as a
   personal skill): coding sessions need it. Redo step 10a whenever a company file or the skill changes
   (merging a company-file PR is not enough for coding sessions);
6. first ticket: something small, with `dry_run: true` in `config.yml` if they want to see the code before
   any PR exists.
Add after the list, once: when the orchestrator restarts it moves its scheduled checks; old ones it cannot
delete are disabled and can be deleted by hand from the scheduled tasks list. Put any warning (unprotected
branch, main-only repo) above the three sections, never inside them.

## /kanbai add-repo <owner/name>
Steps 4–6 for that repo only (and the branch question for it), then: add its section to `repos.md` and its
place in "Coding order"; new data sources to `notes.md`; when screenshots are on and the repo is a UI, add it
to `screenshots.md` (Which repos, plus its Real and Stubbed modes: reuse a fake backend or harness it has,
else stub its API calls in Playwright with `page.route` and a fake login; try it once before writing it); its check commands and branch denies to
`.claude/settings.json`; update `config/company.repos` on the board. Commit on a branch, push, give the PR
link. Then redo step 10a (coding sessions read the company files from the installed skill) and give the
orchestrator restart message with the new repo in its `Attach with push access:` line.

## /kanbai remove-repo <key>
Remove its section and its place in "Coding order", its branch denies and check commands (keep commands
other repos use), and update `config/company.repos`. Tickets in progress that touch it keep their plan; say
which ones. Open PRs in that repo are left alone. Branch, push, PR link, then redo step 10a.

## /kanbai refresh-repos
Steps 4–6 for every repo in `repos.md`, then propose only the differences (new checks, moved folders, new
services, changed default branches) as one PR. Never drop a line the person wrote by hand without asking.
Redo step 10a once the PR is merged.
