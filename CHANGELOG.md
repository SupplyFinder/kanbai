# Changelog

## Unreleased

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
