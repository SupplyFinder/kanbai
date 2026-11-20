# Daily backup and restore (orchestrator)

Two copies, made by the daily reconcile (and by a Health check when today has none):
- **GitHub** (optional): one JSON file per ticket in the private repo `backups.repo` from config.yml,
  branch `backups.branch`. Every daily commit is a dated snapshot; git history keeps them all. When
  `backups.repo` is empty or absent, there is no GitHub copy: skip section 2 (commit = null) and do not
  report it anywhere (not in Health, not in Slack). It is the company's choice: board copy only.
- **The board**: `backup_index/<YYYY-MM-DD>` + `backup_snaps/<T-n@YYYY-MM-DD>` for the last 30 days, which
  the owner's Backups page lists, previews and restores from.

## 1. Collect
For every ticket (archived included): the bundle
`{ ticket, state, approval, replies: [{id, data}], comments: [{id, data}], events: [{id, data}] }`
from `tickets/T-n`, `ticket_state/T-n` (null if none), `approvals/T-n` (null if none),
`tickets/T-n/replies`, `tickets/T-n/comments`, and `events` whose `ticket` is T-n. Also `config/models`,
`config/roles`, `config/branding`, `config/people`, the `people` collection and `approval_votes`.
Fingerprint per ticket: `fp` = `status|plan_rev|status_at|updated_at`, where status is `ticket_state.status`
(or `Submitted`/`Draft` from `submitted_at` when there is no state), `plan_rev` defaults to 0, and
`updated_at` falls back to `created_at`. The board computes the same string to show "changed since".

## 2. GitHub copy
1. `backups.repo` empty or absent: skip to step 3 silently. Otherwise `add_repo` with its owner and name,
   `access: "push"`. If it does not exist or is refused (a repo is named, so this is a mistake to fix): skip to step 3, add the problem to Health ("Backup repo missing: create the private
   repo <backups.repo>"), and tell the owner once in Slack.
2. Clone it to `/home/user/<backup repo name>` (reuse an existing clone; `git -C … fetch
   origin`). Use branch `backups.branch`: `git -C … checkout <branch>` (or `git -C … checkout -b <branch>`
   when it does not exist yet). Write `backup/tickets/T-n.json` (one bundle per file, 2-space JSON, keys sorted),
   `backup/config/{models,roles,branding,people}.json`, `backup/people.json`, `backup/approval_votes.json`,
   and `backup/index.json` = `{date, tickets: {T-n: {title, status, fp}}}`. Keep files of deleted tickets
   (they are the only copy). Attachments and screenshots are not copied (they stay in the board's file
   storage, which deleting a ticket no longer clears).
3. `git -C … add -A`; if anything changed, `git -C … commit -m "Backup <YYYY-MM-DD>"` and
   `git -C … push origin <backups.branch>`. Never push any other branch, never force-push. Note the commit sha.

## 3. Board copy (last 30 days)
1. Read the latest `backup_index` document (not a `safety-` one). For each ticket: if its `fp` equals the
   one there, reuse that `snap` id; otherwise write `backup_snaps/T-n@<today>` =
   `{ticket: "T-n", taken_at: now, bundle}`. If a bundle is too large to write, keep the GitHub copy only,
   set `snap: null` for it and add a Health problem.
2. Write `backup_index/<today>` = `{date: today, at: now, commit: <sha or null>, tickets: {T-n: {title,
   status, fp, snap}}}` (tickets deleted since are simply absent: the page shows them from older days).
3. Prune: delete `backup_index` documents (safety copies included) older than 30 days, then every
   `backup_snaps` document no remaining index references.
4. Write `config/runtime.health.last_backup_at` / `last_backup_commit`.

## Restoring
The owner restores from the board's Backups page; the page writes the documents itself and sends
`Restored … from the backup of …` (see orchestrator.md). For a day older than 30 days, the owner asks the
orchestrator ("restore T-n from <date>"): read `backup/tickets/T-n.json` at the last commit on or before
that date (`git -C … log --before=…`, `git -C … show <sha>:backup/tickets/T-n.json`), write it as
`backup_snaps/T-n@github-<date>` and `backup_index/github-<date>` = `{date, label: "From GitHub, <date>",
at: now, tickets: {T-n: {title, status, fp, snap}}}`, then tell the owner to restore it from the Backups page.
Only the owner may ask for a restore.
