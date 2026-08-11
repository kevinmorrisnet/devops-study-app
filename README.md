# DevOps Study Tracker

A small Python app, two services, for logging study sessions against certification tags - Kubernetes, Terraform, GitOps. Built while following a community DevOps masterclass course, this is not an original project.

## Why this exists

The app itself is deliberately simple. The real point is the pipeline around it, pre-commit hooks, conventional commits, Ruff linting and formatting, pytest with a coverage gate, and a path-filtered GitHub Actions workflow, all built up step by step as the course progresses. The app just needs to be real enough to make those checks meaningful.

## Tech stack

Backend: FastAPI + Uvicorn. Frontend: Flask. Dependency management: uv. Dev environment: mise + devcontainer. Linting/formatting: Ruff. Commit hygiene: commitizen + pre-commit. Testing: pytest, pytest-cov, pytest-asyncio. CI: GitHub Actions. Containers: Docker (multi-stage, non-root, Alpine).

## Architecture

Two independent Python services, no shared code:

- **Backend** (`src/backend`): FastAPI service that stores sessions as CSV rows and serves stats
- **Frontend** (`src/frontend`): Flask app that renders a form and session list, talking to the backend over HTTP

```
frontend (Flask, :22111)  --HTTP-->  backend (FastAPI, :22112)  -->  sessions.csv
```

Each service has its own `pyproject.toml`, `uv.lock`, and `Dockerfile`, and builds independently.

## Project structure

```
.devcontainer/         Dockerfile + devcontainer.json for a consistent dev environment
.github/workflows/     backend-tests.yaml (lint, test, coverage)
scripts/                setup and setup_project (mise trust/install, pre-commit install)
src/backend/            FastAPI service (models, config, storage, tests)
src/frontend/           Flask service (routes, templates)
mise.toml               tool versions and project env
.pre-commit-config.yaml commitizen + ruff hooks
```

## Backend API

5 REST endpoints, session create/list plus a stats aggregate and a health check. FastAPI generates interactive docs automatically, run the backend and open `http://localhost:22112/docs` (Swagger UI) or `/redoc` for the alternate view.

Sessions persist to a flat CSV rather than a database, that is intentional for a study project.

## Running locally

Built around mise and a devcontainer. Opening it in a devcontainer runs `scripts/setup` automatically (`mise trust` + `mise install`).

Without a devcontainer:

```bash
# backend
cd src/backend
uv sync
uv run study-tracker-api

# frontend, in a second terminal
cd src/frontend
uv sync
uv run study-tracker-frontend
```

Backend defaults to `:22112`, frontend to `:22111` and expects the backend at `API_URL` (default `http://localhost:22112`).

Each service also builds independently with Docker:

```bash
docker build -t study-tracker-backend ./src/backend
docker build -t study-tracker-frontend ./src/frontend
```

## Development workflow

Pre-commit runs Ruff (lint + format) and commitizen (conventional commit messages) on every commit:

```bash
pre-commit install
pre-commit install --hook-type commit-msg
```

The devcontainer setup script does this automatically.

## Testing and CI

```bash
cd src/backend
uv run pytest tests/ -v --cov=src/backend --cov-report=xml --cov-fail-under=80
```

The `backend-tests.yaml` workflow only runs when `src/backend/**` changes, via `dorny/paths-filter`, so a frontend-only commit does not trigger a backend test run. On pull requests it also posts a coverage summary as a sticky PR comment, using `irongut/CodeCoverageSummary` and `marocchino/sticky-pull-request-comment`.
