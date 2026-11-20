# Ticket from a conversation

Someone (the owner or a teammate) asks, in any Claude session, to turn the discussion into a ticket on the AI
tickets board (`board_url` in the company `config.yml`; see SKILL.md for where to find it). You write ONE ticket document, as them, with
`ArtifactData` (load it with ToolSearch if it is only listed by name), always as a **Draft**. You never submit
it, plan, code, start sessions or message the orchestrator: submitting is one click on the board, after they
have checked the ticket.

## 1. Draft it from the conversation
Use only what was actually discussed; do not invent requirements. Write in English.
- `title`: one line, max 140 characters, says the change ("Add a category filter to the domain export").
- `type`: `feature`, `bug` or `improvement`. `priority`: `low`, `medium` or `high` (medium unless said).
- `description` (markdown), with the headings that have content:
  `## Context` (who needs it, why) · `## What to change` · `## Expected result` (how we know it is done) ·
  `## Decisions already taken` · `## Out of scope` · `## Open questions` ·
  last line: `_Created from a Claude conversation._`
  Mention repos, screens, endpoints, tables or files only if the conversation named them.
- `screenshots`: false unless they asked for screenshots or the change is clearly visual and they agree.
- Never put secrets, tokens, customer names or customer data (rows from any data source) in the ticket.
- Files from the conversation (screenshots, PDFs, CSV, text, Markdown, JSON that exist as files in this
  session) are attached when possible (step 3). List them in the draft you show.

## 2. Show it and ask before writing
Show the title, type, priority, screenshots flag, files to attach and description, then ask
(AskUserQuestion when available): **Save as draft** · **Change something** (revise and ask again).
Never offer to submit, even if asked: explain that they submit it with one click on the board.
Write nothing until they choose.

## 3. Write it
1. Their board user id: `ArtifactData` `list` on collection `data/users/me` (limit 1); the result says what
   `me` resolved to (`u_…`). That is `author_id`.
2. Number: `get` `meta/counter` (note `next` and `version`) and `list` `tickets` (ids only); n =
   max(`next`, highest used number + 1).
3. Attachments: upload the conversation's files to the board's assets with the `Artifact` tool
   (`action: "publish"`, `url` = board, `asset: true`, `file_paths` = the files; a CSV, Markdown, JSON or
   text file goes in a call of its own with `file_path`). For each upload that succeeds, add
   `{id, name, type}` to `attachments` (id = the asset id from the result, name = the file name, type = its
   MIME type). If uploading is refused (only people with edit rights on the board page can upload: usually
   the owner), attach nothing and tell them which files to add by editing the ticket on the board.
4. One `batch`, so the number is reserved and the ticket created together:
   - `update` `meta`/`counter` `{next: n + 1}` with `if_version` = the version you read;
   - `set` `tickets`/`T-n` WITHOUT `if_version` (so it fails if T-n already exists):
     `{id, title, description, type, priority, author_id, attachments, screenshots, source: "chat",
     created_at: now, submitted_at: "", updated_at: now, updated_by: <their id>}` (always a draft).
   - add `logs/<their id>/entries/<new id>` = `{ticket: "T-n", action: "created as draft from a Claude
     conversation", detail: <title>, at: now}` (their own log, so the board's history shows who created it).
   On a version conflict or "already exists", re-read and retry with the next number (up to 5 times).
5. Reply with the ticket id, the board link, and: "It's a draft: open T-n on the board, check it (Edit to
   change it) and click Submit to Claude when it looks right." 

## If something fails
- No access to the board's data (not shared with them, or view-only): tell them to ask the owner for edit
  access, and give them the drafted text so they can paste it into "New ticket" on the board.
- Never write `ticket_state`, `approvals`, `events` or `config`.
