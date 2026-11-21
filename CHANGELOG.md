# Changelog

## Unreleased

- Setup asks far fewer questions: only which repos to use and who gets the chat skill (plus branches
  when they cannot be detected). No review to confirm and no Slack channel question: company name,
  ticket prefix (`KB` by default), time zone, Slack (off), screenshots, backups and logo take defaults
  shown in the closing summary, editable in `kanbai/config.yml` or the board's Settings → Branding.
- Setup ends with three sections: Connectors, a "Defaults used" table (setting, value, where to change),
  and What's left for you.
- The orchestrator attaches only the product repos named in the message that starts it
  (`Attach with push access: …`, given ready to paste by setup); a missing repo is a Health problem that
  says how to fix it.
- Fix: coding sessions were refused when attaching product repos with push access. The skill's own text
  (coding, orchestrator and setup instructions, and the "Attach with push access" lines the orchestrator
  added to each session's first message) said that attaching a repo needs a person to name it; the session
  reads that text before attaching, and its permission check then refused the attach. Those sentences and
  lines are gone: ticket sessions start with only `/kanbai <stage> T-n`, `board:` and `orchestrator:`, and
  attach the repos in `repos.md` as part of their task. A failed attach still pauses the ticket.
- Setup stops watching the board right after publishing it, so the first submitted ticket wakes the
  orchestrator, not the setup session.
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
