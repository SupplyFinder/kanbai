# Setup and repo commands

Run by a person in a Claude Code session started on the company repo (a copy of the KanbAI template).
Commands: `/kanbai setup`, `/kanbai add-repo <owner/name>`, `/kanbai remove-repo <key>`,
`/kanbai refresh-repos`. They only ever change files of the company repo (through a branch and a PR)
and the board's settings documents; never a product repo.

**Ask as little as possible.** Ask only what is mandatory and cannot be detected or defaulted: which repos
the pipeline may work on (step 3), and one final review of the drafts (step 8). Ask about branches
(step 5) only when detection is unclear. Everything else takes the default below, is shown in the step 8
review marked "default", and can be changed there or later in `kanbai/config.yml` (the orchestrator picks it up
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
- Ticket prefix: 2–4 capitals from the company name. Never one already used by another pipeline: if the
  picked repos have branches or PRs named `…/<PREFIX>-<number>-…`, or another KanbAI/AI-tickets company repo
  uses it, take the next free one (e.g. `KB`). Changing it later only affects new tickets.
- Time zone: from the person's locale or email domain country; else `UTC`.
- Slack: off, unless `SLACK_BOT_TOKEN` is set in this environment; then ask only for the channel id (step 8).
- PR screenshots: off (the author can still tick it per ticket once it is enabled in `config.yml`).
- Backups: board only (30 days); `backups.repo` empty. Say in the review that a private GitHub repo can be
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
them in the step 8 review. Ask only about the repos whose detection is unclear (no PR history, or PRs
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
over a square icon), say which in the review, and that it can be changed in the board's Settings → Branding. Suggest the main colour from the chosen logo. For dark mode, if the logo has dark text,
make a light-text copy. A website the network blocks: say so and fall back to the avatar or none.

### 8. Write the company files
Draft `kanbai/config.yml`, `repos.md`, `notes.md` (data sources found, with "not connected" where so; UI
text; notable changes; environments; then a few questions the code cannot answer: what Claude must never
change without asking, who reviews PRs) and `screenshots.md` (only if enabled: which repos, how to log in,
preview URLs if PR checks show preview deployments). Format: `company-files.md`. For the questions the code
cannot answer, write a default ("Notable changes" list; reviews: the owner) instead of asking.
Show the drafts in one message, marking each line "detected", "default" or "you said", then ask ONE
question: "Looks right" (Recommended) or "Change something" (they type what in Other). In the same call,
only when needed: the Slack channel id (step 2) and the chat skill (step 10a). Apply the edits and go on;
ask again only if an edit is unclear.

### 9. Permission rules
Write `.claude/settings.json` from `settings.base.json` (next to this file; drop its `_note`) plus, for this company:
- allow: each repo's check commands (as `Bash(<command prefix>:*)`), the install commands they need, and
  the data connectors' tools (`mcp__<connector>`);
- deny, for every repo: `Bash(git push origin <PR target>:*)`, `Bash(git push origin <release>:*)`, the same
  with `-u` / `--set-upstream`, force pushes and branch deletions;
- never allow merging PRs or turning on auto-merge.
These rules catch plain `git push` lines only; the real protection is GitHub branch protection (step 4):
list unprotected PR targets and release branches in the step 8 review (no separate question): the board
shows a warning until they are protected on GitHub. If the person says in the review that protection is
not wanted, add `Protection: not required for <branch>` to that repo's section in `repos.md`; the board
then shows a note instead.

### 10. Publish the board
Publish `board/board.html` as a new artifact (`Artifact` publish, `icon: "kanban"`, a one-line
description) with the capabilities in `board/rules.json`. Put its URL in `config.yml` `board_url`. Then
seed its settings with `ArtifactData`: `config/company` (orchestrator.md, step 1), `config/models` (the
defaults in board.md), `config/runtime` (`slack_channel` when Slack is on), `config/branding` (upload the
chosen logo files to the board's assets first, then `{logo_light, logo_dark, accent}`).

### 10a. Chat skill (asked in the step 8 review, not separately)
Who gets "create a ticket in the kanban from our discussion" in any Claude chat:
"The whole organisation (Recommended)", "Only me" or "Not now". For either yes, first build a plugin folder
in the scratchpad (never in the repo): `kanbai-chat/.claude-plugin/plugin.json` = `{"name": "kanbai",
"version": "<skill version>", "description": "Create KanbAI tickets on the <company> board from any Claude
chat."}`, and `kanbai-chat/skills/kanbai/` = every file of this skill folder plus `kanbai/config.yml`
copied next to `SKILL.md` (the skill reads it there outside the repo). No other company file, no secret.
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
link to open a PR (open it yourself only if they ask). Then list what is left for them:
1. merge the PR;
2. share the board with the organisation (the board's Share menu);
3. start the orchestrator in a NEW session on the company repo with this message, filled in (give it
   ready to paste, one repo per `owner/name`, every repo in `repos.md`, none from `forbidden_repos`):
   `/kanbai orchestrator <board url>` and, on the next line, `Attach with push access: <owner/name>, …`.
   The person typing the repos is what lets the session attach them in Auto mode;
4. teammates open the board once; tick approvers and admins in Permissions;
5. if the chat skill was set up in step 10a: approve it in the organisation library if asked (organisation),
   or upload the zip as a personal skill (only me); redo step 10a whenever `config.yml` or the skill changes;
6. first ticket: something small, with `dry_run: true` in `config.yml` if they want to see the code before
   any PR exists.
Also list the defaults used (prefix, time zone, Slack, screenshots, backups, logo) in one line each, with
where to change them. Also say once: when the orchestrator restarts it moves its scheduled checks; old ones it cannot delete are
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
