# HavocHelp — Agent Instructions

## Architecture (Mental Model)

```
Browser (Leaflet map UI)
  └── Django (hintas/)
        ├── /upload_snapshot/   — receives base64 drone images, scores them via OpenAI, stores in Redis + DB
        ├── /get_route/         — returns ranked GPS waypoints from scored snapshots
        └── /                  — renders the single-page map UI (index.html)

Redis                          — ephemeral snapshot scoring during a drone run (not a primary store)
SQLite                         — local persistence (Building, DroneSettings, ListData models)
OpenAI GPT-4o-mini             — vision API: classifies damage level + building size from images
```

## Key Files

- `hintas/settings.py` — Django config. **Has hardcoded SECRET_KEY and DEBUG=True — never commit real secrets.**
- `home/models.py` — Three models: `Building`, `DroneSettings` (singleton, pk=1), `ListData`
- `home/views.py` — All views + API endpoints (`index`, `upload_snapshot`, `get_route`)
- `home/sorting.py` — OpenAI GPT-4o-mini vision call: returns `(damage_class, size)` tuple
- `home/utils/displace.py` — GPS math: bearing + speed + time → new coordinates
- `home/utils/emergency.py` — Haversine-based rescue team assignment
- `home/utils/rebuild.py` — Weighted damage + size + area reconstruction priority score
- `home/utils/order.py` — Greedy resource-constrained rebuild ordering
- `home/templates/index.html` — Leaflet map UI (single page, JS-heavy)
- `pyproject.toml` — uv project config + dependencies

## Development Commands

```bash
uv sync                              # install dependencies
uv run python manage.py runserver    # dev server
uv run python manage.py migrate      # apply migrations
uv run python manage.py makemigrations  # after model changes
uv run python manage.py test         # run tests
uv run python manage.py shell        # Django shell
```

## Git

- Conventional Commits: `feat:`, `fix:`, `chore:`, `refactor:`, etc.
- **Never add a co-author line to commit messages.**
- Before committing, always check the current branch. Commit to a relevant branch (`feat/...`, `fix/...`, `chore/...` off `main`) with a descriptive name, then push it to the remote.

## Coding Rules

**No secrets in code** — `SECRET_KEY`, `OPENAI_API_KEY`, `REDIS_URL` must come from environment variables. Never hardcode them.

**uv only** — Never use pip directly. Add deps via `uv add <package>`. No `requirements.txt`.

**Keep migrations** — Never delete migration files. Always run `makemigrations` after model changes.

**Fail fast** — Missing config at startup = immediate error, not silent fallback.

**Minimal changes** — Only change what's needed for the task. Don't refactor unrelated code.

**Read before editing** — Always read a file before modifying it.

**One concern per PR** — Keep pull requests focused. Don't bundle unrelated changes.

**Document new endpoints** — When adding URL endpoints, add them to the README.

## Known Issues (do not fix unless specifically tasked)

- `hintas/settings.py`: `SECRET_KEY` is hardcoded and `DEBUG=True` — needs env var migration before production use
- `home/sorting.py`: `api_key` variable is referenced but must be injected from `settings.OPENAI_API_KEY`
