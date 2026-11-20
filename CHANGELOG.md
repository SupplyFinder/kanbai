# Changelog

## Unreleased

- Setup asks far fewer questions: only which repos to use and one review of the drafts (plus branches
  when they cannot be detected). Company name, ticket prefix (never one another pipeline already uses),
  time zone, Slack, screenshots, backups and logo take defaults shown in the review, editable in
  `kanbai/config.yml` or the board's Settings → Branding.
- The orchestrator attaches only the product repos named in the message that starts it
  (`Attach with push access: …`, given ready to paste by setup). In Auto mode an unnamed repo could be
  refused even though `add_repo` is allowed; a missing repo is now a Health problem that says how to fix it.
- Ticket sessions are told in their first message which repos they may attach (push for coding, read
  for planning and review). Auto mode judges `add_repo` from the session's own messages, so an attach
  was sometimes refused; a coding session now pauses instead of retrying when that happens.
- The hourly check's message names the KanbAI board (it said "AI tickets board").
- Setup asks whether to make the chat skill available to the whole organisation (published as a
  `kanbai` plugin) or only to you (a zip to upload as a personal skill).
- Board: with no tickets yet, the stage columns still show, with the empty-board message below them.
- The backup repo is optional: leave `backups.repo` empty to keep the board copy only (30 days).
  Setup asks; Health no longer reports a missing repo when none is configured.
- Orchestrator restart only takes over scheduled checks of this board's previous orchestrator.

## 0.1.0 (first public release)
- Kanban board (a claude.ai artifact): tickets, plans, approvals, PR status, comments, change log,
  search and filters, ticket links, reports, roles (submitter, approver, admin), usage bar and hold,
  pipeline health, daily backups with restore, branding, light and dark themes.
- Skill `kanbai`: orchestrator, planning, coding (draft PRs), optional Claude review, tickets from a
  conversation, daily backup to a private repo, setup and repo commands.
- Company settings in `kanbai/` (config, repos, notes, screenshots), written by `/kanbai setup`.
