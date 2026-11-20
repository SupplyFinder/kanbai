# Screenshots for UI PRs

Only when `config.yml` has `screenshots.enabled: true` AND the ticket's `screenshots` box is ticked
(`tickets/T-n.screenshots` true), or the owner asks for screenshots in a PR or board comment. Then every PR
that changes something a user can see (a screen, a layout, a state, a label or any visible text) in a repo
the company `screenshots.md` lists gets at least one screenshot of the changed screen, at most 4. Skip
backend-only work, refactors with no visible change, and changes you cannot render honestly (then say why
in the PR). How to log in, reach a preview or run the app, and stub the backend: the company
`kanbai/screenshots.md`.

## Pick the mode

| The UI change depends on… | Mode | What the reviewer sees |
| --- | --- | --- |
| Nothing new in the backend (UI-only, or the API it calls is already deployed) | **Real** | Real data of the test account |
| A backend change in this ticket (not deployed yet) | **Stubbed** | Made-up fixture data |

A deployed test environment only has what is merged and deployed: a new endpoint from this ticket does not
exist there, so never use Real mode for it. Capture at 1440×900, `deviceScaleFactor: 2`. Write any
throwaway script OUTSIDE the repo (e.g. `/tmp/shots/`). Revert any test-harness change before committing,
unless the plan asked for tests. Never commit images into the product repo.

## Never in a screenshot
Real customer names, emails or data that is not the test account's own, tokens, or the browser's dev
tools. Crop or skip when unsure. Plus what the company `screenshots.md` lists.

## Where screenshots go (both)
1. **The PR.** Commit the PNGs to `config.yml` `screenshots.branch` of `screenshots.repo` under
   `T-n/<key>/<name>.png` (create the branch if missing; never touch another branch). Add a
   `## Screenshots` section to the PR body (`update_pull_request` if the PR is already open): one line per
   image saying what it shows and the mode, e.g. `Dashboard greeting (real data)`, then
   `![alt](https://github.com/<screenshots.repo>/blob/<screenshots.branch>/T-n/<key>/name.png?raw=true)`.
2. **The board** (shown in the ticket's Code tab). Upload each PNG to the board's asset store with the
   `Artifact` tool: `action: "publish"`, `url` = the board URL, `asset: true`, `file_path` (or `file_paths`
   for several). Then append to `ticket_state/T-n.screenshots` (read it first, pin `if_version`):
   `{ "asset_id": "<id from the upload>", "caption": "<what it shows>", "mode": "real" | "stubbed", "side": "<key>", "pr": <PR number> }`.

When the owner comments on a UI PR asking for (new) screenshots, for example once its backend PR is
deployed, take them in Real mode and add them the same way.
