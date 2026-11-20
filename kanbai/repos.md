# Repos

Written by `/kanbai setup` (and `add-repo`, `remove-repo`, `refresh-repos`). Format:
`.claude/skills/kanbai/company-files.md`. Example below; setup replaces it.

## Coding order
`api`, then `web`.

## api
- Repo: `acme/acme-api` · Label: API
- What it is: customer API (FastAPI)
- PR target: `dev` · Release branch: `main` · Branches: `feature/T-n-<slug>`
- Checks (mirror CI): `ruff check .`, `pytest`
- Layout: routes in `app/routes/`, logic in `app/services/`, models in `app/models/`.

## web
- Repo: `acme/acme-web` · Label: Web app
- What it is: customer UI (React + Vite), built against `api`
- PR target: `dev` · Release branch: `main` · Branches: `feature/T-n-<slug>`
- Depends on: `api` (attach it read-only when coding `web`)
- Checks (mirror CI): `npm run lint`, `npm run typecheck`
- Layout: pages in `src/pages/`, API calls in `src/services/`. UI text rules in notes.md.
