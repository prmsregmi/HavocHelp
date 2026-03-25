# HavocHelp

AI-powered disaster damage assessment and emergency response coordination system. HavocHelp processes drone-captured imagery using OpenAI Vision to classify building damage, calculate rescue priorities, and optimize team routing for emergency response and reconstruction planning.

## What It Does

- **Damage Classification**: Analyzes drone images with GPT-4o-mini to classify building damage (none, mild, moderate, severe, catastrophic) and building size
- **Emergency Routing**: Distributes rescue teams across affected areas using GPS-based prioritization and Haversine distance calculations
- **Reconstruction Planning**: Scores and orders buildings for repair based on damage severity, size impact, and resource constraints
- **Live Drone Simulation**: Simulates drone flight paths with real-time GPS coordinate calculation, snapshotting and scoring locations along the route
- **Interactive Map**: Leaflet-based map UI to visualize drone paths, building markers, and team assignments

## Tech Stack

- **Backend**: Django 5.2, Python 3.13
- **AI**: OpenAI GPT-4o-mini (vision) for damage/size classification
- **Caching**: Redis for in-flight snapshot scoring
- **Package Management**: [uv](https://docs.astral.sh/uv/)
- **Server**: Gunicorn

## Getting Started

### Prerequisites

- Python 3.13+
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- Redis server running locally
- OpenAI API key

### Setup

```bash
# Clone the repo
git clone https://github.com/prmsregmi/HavocHelp.git
cd HavocHelp

# Install dependencies
uv sync

# Set environment variables
cp .env.example .env
# Edit .env and add your OPENAI_API_KEY and SECRET_KEY

# Run migrations
uv run python manage.py migrate

# Start the dev server
uv run python manage.py runserver
```

### Environment Variables

| Variable | Description |
|---|---|
| `SECRET_KEY` | Django secret key |
| `OPENAI_API_KEY` | OpenAI API key for image analysis |
| `REDIS_URL` | Redis connection URL (default: `redis://localhost:6379`) |
| `DEBUG` | Set to `False` in production |

## How It Works

1. Configure drone starting coordinates, speed, and direction via the settings panel
2. The drone simulation sends snapshots (base64 images) to `/upload_snapshot/`
3. Each image is analyzed by GPT-4o-mini to return a damage class and building size
4. Scored results are cached in Redis and persisted in the database
5. `/get_route/` returns the optimal GPS waypoints based on combined damage/size/proximity scores
6. Emergency teams are assigned starting points and routed using greedy nearest-neighbor allocation

## Project Structure

```
havochelp/
├── hintas/          # Django project config (settings, urls, wsgi/asgi)
├── home/            # Main app
│   ├── models.py    # Building, DroneSettings, ListData models
│   ├── views.py     # Main views and API endpoints
│   ├── sorting.py   # OpenAI Vision integration
│   ├── templates/   # Leaflet map UI
│   └── utils/
│       ├── displace.py    # GPS displacement calculations
│       ├── emergency.py   # Rescue team routing
│       ├── rebuild.py     # Reconstruction priority scoring
│       └── order.py       # Resource-constrained rebuild ordering
└── static/          # Static assets
```

## License

MIT — see [LICENSE](LICENSE)
