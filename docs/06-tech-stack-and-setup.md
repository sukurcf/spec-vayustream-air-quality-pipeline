# Tech stack and setup

## Approved versions

Use only these verified cohort version families. Pin the PostgreSQL image to a minor release in the required 18.x family.

| Tool | Version | Use |
|---|---|---|
| Python | 3.12.x | Local environment and test helpers |
| uv | 0.12.x | Reproducible Python environment and lockfile |
| dbt Core | 1.12.x | Project parsing, model builds, seeds, and tests |
| dbt-postgres | 1.11.x | PostgreSQL adapter |
| PostgreSQL | 18.x; image `postgres:18.6` | The one local database |
| Docker Engine | 29.x | Run PostgreSQL locally |
| Docker Compose | v5.x (5.5) | Start and stop the database |
| pytest | 9.1.x | Python support-package tests, if present |
| Ruff | 0.16.x | Python lint and format checks, if Python support code is present |
| GitHub Actions | Existing cohort CI service | Run the defined CI job; not a runtime dependency |

The required project is dbt and SQL, not a Python application. Do not add another framework, database, API client, or orchestration service.

## Local configuration

| Setting | Required value | Handling |
|---|---|---|
| PostgreSQL host | `127.0.0.1` | Bind only to loopback |
| PostgreSQL host port | `15433` | Compose maps this to the container's PostgreSQL port |
| Database | `dbt_starter` | Local project database |
| dbt target | `local` | Reads local connection settings |
| Credentials | Local environment values | Keep real values out of Git; examples use placeholders |

The Compose project starts only PostgreSQL. Keep generated database data and dbt artifacts local and untracked.

## Reference hardware

Use a local laptop with 4 CPU cores, 8 GB RAM, and at least 5 GB free disk as the proposed demo target. These are acceptance conditions, not measured performance claims.

## Windows setup

On Windows, run the project from a local WSL2 Ubuntu environment with Docker Desktop's WSL integration enabled. Keep the student checkout inside the WSL filesystem. macOS and Linux users can use Docker Compose with their local Docker engine.

## Setup and offline boundary

1. Install Python 3.12, uv 0.12.x, Docker Engine 29.x, and Docker Compose v5.x.
2. In the separate student implementation repository, resolve and lock dbt Core 1.12.x and dbt-postgres 1.11.x with uv.
3. Download the locked Python dependencies and the pinned PostgreSQL image while internet is available.
4. Configure the local dbt target and placeholder-backed PostgreSQL credentials.
5. After preparation, block external network access and use only the local database and fictional fixture.

Do not silently switch to a different database or source if PostgreSQL is unavailable.

## Required demo commands

Run from the student implementation repository:

```text
docker compose up -d db
uv run dbt seed --target local
uv run dbt build --target local --full-refresh
uv run dbt test --target local
```

The first command starts PostgreSQL on `127.0.0.1:15433`. The next command seeds the two source tables. The full-refresh build recreates the required transformations, and the test command reports every configured dbt test. To stop PostgreSQL without deleting its local volume, run `docker compose down`.

## Artifacts and documentation

Local dbt artifacts such as parse/build result files may be kept in the student repository's ignored output directory. Generating or inspecting local artifacts is acceptable. Do not host, publish, or style a documentation website, and do not add a hosted dependency to the demo.

[Back to README](../README.md)
