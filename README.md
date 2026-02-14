<div align="center">

# 🏎️ F1 Performance Analytics Platform

**Real-time Formula 1 lap performance analysis powered by FastF1**

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)](https://python.org)
[![Django](https://img.shields.io/badge/Django-5.1-092E20?logo=django&logoColor=white)](https://djangoproject.com)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://postgresql.org)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![FastF1](https://img.shields.io/badge/FastF1-3.4-E10600?logo=f1&logoColor=white)](https://docs.fastf1.dev)

Ingest F1 session data • Store in PostgreSQL • Analyse via REST API • Visualise in React

---

</div>

## Dashboard — Driver Comparison

![Dashboard Preview](docs/dashboard-preview.svg)

> Compare lap-by-lap performance for any two drivers. Stat cards show best/average lap times, sector splits, and tyre compound breakdown. Interactive charts powered by Recharts.

## Ingestion — Full Season Import

![Ingestion Preview](docs/ingest-preview.svg)

> Ingest a single GP or an entire season with one click. Live progress bars track each event as FastF1 fetches and caches telemetry data.

---

## Features

🔴 **Single GP or Full Season ingestion** — fetch one session or every race in a year with one click

📊 **Interactive dashboard** — lap time trends, driver comparison, sector analysis, compound breakdown

⚡ **FastF1 caching** — first fetch downloads raw data; subsequent loads are near-instant from a Docker volume

🔌 **REST API** — 8 endpoints powering the frontend and available for your own scripts or notebooks

🐳 **One command to run** — `docker compose up --build` starts Postgres, Django, and React together

---

## Quick Start

```bash
git clone https://github.com/YOUR_USERNAME/f1-analytics.git
cd f1-analytics
docker compose up --build
```

| Service  | URL                          |
|----------|------------------------------|
| Frontend | http://localhost:5173        |
| API      | http://localhost:8000/api    |
| Postgres | `localhost:5432`             |

### Ingest your first race

Open http://localhost:5173 → enter **Year:** `2024`, **GP:** `Bahrain`, **Session:** `R` → click **Ingest Data**.

The first fetch takes 1–3 minutes (FastF1 is downloading ~100 MB of session data). After that, the same session loads in seconds from cache.

### Ingest a full season

Switch to the **Full Season** tab → select year and session types (`Q`, `R`, etc.) → click **Ingest Full Season**. Progress updates live as each GP completes.

---

## Architecture

```
┌─────────────────┐       ┌────────────────────────┐       ┌──────────────┐
│   React + Vite  │──────▶│  Django REST Framework  │──────▶│  PostgreSQL  │
│   :5173         │  API  │  + FastF1 ingestion     │  SQL  │  :5432       │
│                 │◀──────│  :8000                  │◀──────│              │
└─────────────────┘       └────────────────────────┘       └──────────────┘
                                    │
                                    ▼
                           ┌─────────────────┐
                           │  FastF1 Cache    │
                           │  (Docker Volume) │
                           └─────────────────┘
```

---

## API Reference

### Ingestion

```bash
# Single session
curl -X POST http://localhost:8000/api/ingest \
  -H "Content-Type: application/json" \
  -d '{"year": 2024, "gp": "Bahrain", "session": "R"}'

# Full season (qualifying + race for all GPs)
curl -X POST http://localhost:8000/api/season/ingest \
  -H "Content-Type: application/json" \
  -d '{"year": 2024, "sessions": ["Q", "R"]}'

# Preview a season calendar
curl "http://localhost:8000/api/season/schedule?year=2024"

# List all ingestion runs
curl http://localhost:8000/api/ingest
```

### Data & Analytics

```bash
# Drivers in a session
curl "http://localhost:8000/api/drivers?ingestion_run_id=1"

# Lap times with filters
curl "http://localhost:8000/api/lap-times?ingestion_run_id=1&driver=VER&compound=MEDIUM"

# Performance summary
curl "http://localhost:8000/api/performance?ingestion_run_id=1&driver=VER"

# Compare two drivers
curl "http://localhost:8000/api/performance/compare?ingestion_run_id=1&drivers=VER,NOR"
```

<details>
<summary><strong>Full endpoint list</strong></summary>

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/ingest` | Trigger single session ingestion |
| `GET` | `/api/ingest` | List all ingestion runs |
| `GET` | `/api/ingest/:id` | Ingestion run detail |
| `GET` | `/api/season/schedule?year=` | Preview season calendar |
| `POST` | `/api/season/ingest` | Trigger full season ingestion |
| `GET` | `/api/season/ingest` | List season ingestion jobs |
| `GET` | `/api/season/ingest/:id` | Season job detail + progress |
| `GET` | `/api/drivers?ingestion_run_id=` | Drivers in a session |
| `GET` | `/api/lap-times?ingestion_run_id=&driver=` | Lap times (filterable) |
| `GET` | `/api/performance?ingestion_run_id=&driver=` | Aggregated stats + trend |
| `GET` | `/api/performance/compare?ingestion_run_id=&drivers=` | Multi-driver comparison |

</details>

---

## Project Structure

```
f1-analytics/
├── docker-compose.yml          # 3 services: db, backend, frontend
├── .env.example                # Environment variable template
├── .gitignore
├── README.md
│
├── docs/
│   ├── dashboard-preview.svg   # Dashboard screenshot
│   └── ingest-preview.svg      # Ingestion page screenshot
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── manage.py
│   ├── core/
│   │   ├── settings.py         # Django config + FastF1 cache dir
│   │   ├── urls.py
│   │   └── wsgi.py
│   └── api/
│       ├── models.py           # IngestionRun, SeasonIngest, Driver, LapTime
│       ├── serializers.py      # DRF serializers
│       ├── views.py            # All REST endpoints
│       ├── ingest_service.py   # FastF1 fetch + DB load logic
│       ├── urls.py             # API routing
│       └── admin.py
│
└── frontend/
    ├── Dockerfile
    ├── package.json
    ├── vite.config.js          # Dev server + API proxy
    ├── index.html
    └── src/
        ├── main.jsx            # App shell + routing
        ├── index.css           # Dark theme design system
        ├── api/
        │   └── client.js       # Axios API client
        └── pages/
            ├── IngestPage.jsx  # Single GP + Full Season tabs
            └── DashboardPage.jsx  # Charts, stats, comparison
```

---

## Database Schema

```
┌──────────────────┐     ┌──────────────┐     ┌──────────────────┐
│  IngestionRun    │     │   Driver     │     │   SeasonIngest   │
├──────────────────┤     ├──────────────┤     ├──────────────────┤
│  id              │     │  id          │     │  id              │
│  year            │     │  driver_code │     │  year            │
│  gp_name         │     │  driver_name │     │  session_types   │
│  session_type    │     │  team_name   │     │  status          │
│  status          │     └──────┬───────┘     │  total_events    │
│  driver_count    │            │              │  completed_events│
│  lap_count       │            │              │  failed_events   │
│  started_at      │     ┌──────┴───────┐     │  error_log       │
│  finished_at     │     │   LapTime    │     │  started_at      │
└──────┬───────────┘     ├──────────────┤     │  finished_at     │
       │                 │  id          │     └──────────────────┘
       └────────────────▶│  ingestion_run│
                         │  driver       │
                         │  lap_number   │
                         │  lap_time_ms  │
                         │  sector_1_ms  │
                         │  sector_2_ms  │
                         │  sector_3_ms  │
                         │  compound     │
                         │  track_status │
                         │  created_at   │
                         └──────────────┘
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **FastF1 cache errors** | `docker compose down -v && docker compose up --build` to reset volumes |
| **First ingestion is slow** | Normal — FastF1 downloads 50–200 MB per session on first fetch. Cached after that. |
| **Database connection errors** | `docker compose restart backend` — the health check should handle startup order |
| **Frontend can't reach API** | Vite proxies `/api` to the backend. Outside Docker, set `VITE_API_URL=http://localhost:8000` |
| **Some GPs fail in season ingest** | Expected for cancelled events or testing sessions. Failures are logged without stopping the batch. |

### Supported Data

- **Years:** 2018 onwards (FastF1 coverage)
- **Sessions:** `FP1`, `FP2`, `FP3`, `Q` (qualifying), `R` (race), `S` / `SQ` (sprint, year-dependent)
- **GP names:** Full names (`"Monaco Grand Prix"`), short names (`"Monaco"`), countries (`"Bahrain"`), or round numbers (`1`)

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data Source** | FastF1 | F1 telemetry & lap data via the Ergast API |
| **Database** | PostgreSQL 16 | Persistent storage for all session data |
| **Backend** | Django 5.1 + DRF | REST API, ingestion orchestration, analytics queries |
| **Frontend** | React 18 + Vite | Dashboard UI with interactive charts |
| **Charts** | Recharts | Lap time trends and driver comparison |
| **Infra** | Docker Compose | Single-command local deployment |

---

## License

MIT

