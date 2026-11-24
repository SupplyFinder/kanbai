# Changelog

## Unreleased

- Blocking tickets are honoured from planning on (0.2.2). A submitted ticket with an open blocker now waits
  (status `Submitted`, with a note shown on the ticket) and is planned only once the blocker is Done, instead
  of being planned against code the blocker had not merged yet. Planning reads each blocker's ticket, brief,
  plan and merged PRs, builds on them, and lists in the brief where its ticket describes the blocker's work
  differently from the merged code (following the code; asking only when it changes the outcome). Coding
  reads the blockers' plans and PRs for context only. The board shows the waiting note on `Submitted` and
  `Queued` tickets.
- Planning reads the PR target branch (0.2.1). A planning session's clone landed on the repo's default
  branch, often the release branch, so a ticket planned after its blocker was merged still planned against
  code without the blocker's work. Planning now checks out each repo's PR target after cloning, and a coding
  session does the same for a read-only "Depends on" repo.
- Coding sessions start on the product repo (0.2.0). Attaching a repo with push access from a session
  started on the company repo is decided by a permission check that refused it again and again. The
  orchestrator now starts one coding session per repo in scope, on that repo (`source_url`, `outcome_branch`
  = the ticket branch), one after the other in coding order; nothing is attached with push access. The
  session loads `/kanbai` and the company files from the skill installed on the account, so setup step 10a
  is required and its copy carries all four company files (`config.yml`, `repos.md`, `notes.md`,
  `screenshots.md`); redo it whenever a company file or the skill changes. `ticket_state` gains
  `code_sessions` and `coding_key`. A comment for another repo is forwarded to that repo's session.
- Auto-approve: the owner, approvers and admins can tick "Auto-approve the plan" on a ticket (form or Request
  tab). It is their own `approval_votes` entry (`auto: true`); every plan revision is then approved and coding
  starts without a click. The orchestrator checks the role again.
- Drag and drop on the board, by role: Draft → With Claude (submit), Plan ready → Coding (approve),
  Needs info → With Claude (go ahead without more answers), Paused → its stage (Continue), any column → Done.
  Columns a card cannot go to turn grey while dragging. Mark done is also a button.
- Mark done: the orchestrator stops the ticket's sessions, closes its unmerged PRs (merged ones are kept) and
  sets Done. The request is the person's own `approval_votes` entry (`done: true`).
- Claude at work is visible: pulsing dots on the status pill while a session runs ("Approved, coding starts"
  too); a comment sent to Claude, answers, plan changes or Continue move the card to its working column at
  once (`claude_pending_at`, ended when the status changes, not by clock).
- Comments: the post button says where the comment goes ("Post and send to planning / coding"); cards show
  "Comment sent", "Claude needs details" or "Claude replied"; the Plan or Code tab shows the last update made
  after a comment.
- Submit, Approve, Continue and Send answers close the ticket; archive and delete work at any status (the
  orchestrator stops the sessions first).
- A submitted ticket Claude was never told about shows a "Send to Claude now" banner.
- Health: "Run a check now" shows Running with dots until the new report arrives. Health, Reports, Backups,
  Permissions and Settings close on a click outside. Fix: a stray "null" in Health.
- Fix: dropping a card on Done did nothing (the browser confirm is blocked in the claude.ai frame); the board
  uses its own confirmation.
- Fix: coding sessions refused to attach repos. The orchestrator passes `add_repo` and `register_repo_root` to
  the sessions it starts (`create_session` `extra_allowed_tools`), and a refused attach no longer tells people
  to edit `settings.json` (it already allows `add_repo`).
- Sessions write real timestamps (`date -u`), never rounded or estimated ones.
- `/kanbai add-repo` sets up screenshots for a new UI repo (its own harness, or the API stubbed in
  Playwright with a fake login, tried once before it is written down).
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
