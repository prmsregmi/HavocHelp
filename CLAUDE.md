# CLAUDE.md — HavocHelp

This file provides context and guidelines for AI assistants (Claude Code, Copilot, etc.) working in this repository.

## Project Overview

HavocHelp is a Django web application that uses OpenAI Vision to analyze drone imagery for disaster damage assessment and emergency response coordination. It classifies building damage, routes rescue teams, and prioritizes reconstruction using GPS-based scoring algorithms.

## Development Commands

```bash
# Install dependencies
uv sync

# Run development server
uv run python manage.py runserver

# Run migrations
uv run python manage.py migrate

# Make new migrations
uv run python manage.py makemigrations

# Django shell
uv run python manage.py shell

# Run tests
uv run python manage.py test

# Open Django admin
# Visit http://localhost:8000/admin/
```

## Architecture

- **`hintas/`** — Django project config. Settings, URLs, WSGI/ASGI entrypoints.
- **`home/`** — The single Django app with all domain logic.
  - `models.py` — Three models: `Building`, `DroneSettings` (singleton), `ListData`
  - `views.py` — Main page render, `upload_snapshot` API, `get_route` API
  - `sorting.py` — OpenAI GPT-4o-mini vision call for damage/size classification
  - `utils/displace.py` — GPS offset math (bearing + speed + time → new coords)
  - `utils/emergency.py` — Rescue team assignment using Haversine distance
  - `utils/rebuild.py` — Weighted damage + size + area reconstruction priority
  - `utils/order.py` — Greedy resource-constrained rebuild ordering
  - `templates/index.html` — Leaflet map UI (JS-heavy, single page)

## Key Conventions

- Use `uv` for all dependency management. Never use pip directly. Do not add a `requirements.txt`.
- Python 3.13+. Use modern Python idioms (match/case, walrus operator, type hints where helpful).
- Environment variables via `.env` file (not committed). All secrets (`SECRET_KEY`, `OPENAI_API_KEY`) must come from env.
- Keep Django settings `DEBUG=False` and `SECRET_KEY` out of source. The current `settings.py` has hardcoded values — move them to env before any production use.
- Redis is used for ephemeral snapshot scoring during a drone run. It is not a primary store.
- SQLite is used locally. Do not commit `db.sqlite3`.

## AI Assistant Rules

1. **Read before editing.** Always read a file before modifying it.
2. **Minimal changes.** Only change what's needed for the task. Don't refactor unrelated code.
3. **No secrets in code.** Never hardcode API keys, passwords, or secret keys.
4. **Test your logic.** For scoring/routing algorithms, verify with concrete examples before committing.
5. **Keep migrations.** Do not delete existing migration files. Always run `makemigrations` after model changes.
6. **uv only.** Add dependencies via `uv add <package>`, not by editing `pyproject.toml` manually unless necessary.
7. **One concern per PR.** Keep pull requests focused. Don't bundle unrelated changes.
8. **Document APIs.** When adding new URL endpoints, document them in the README.

## Environment Setup

Copy `.env.example` to `.env` and fill in:

```
SECRET_KEY=your-django-secret-key
OPENAI_API_KEY=sk-...
DEBUG=True
REDIS_URL=redis://localhost:6379
```
