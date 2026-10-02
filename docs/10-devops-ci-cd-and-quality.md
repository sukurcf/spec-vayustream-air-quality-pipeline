# DevOps, CI/CD, and quality

Purpose: This document defines the VayuStream engineering workflow, CI gates, configuration, and Definition of Done.

## Git workflow

Students MUST use GitHub Flow.

| Rule | VayuStream requirement |
|---|---|
| Main branch | `main` always contains runnable lite profile. |
| Work branches | Create one branch per issue or feature. |
| Pull requests | Every change uses a PR. No direct push to `main`. |
| PR size | Prefer small PRs under 400 changed lines. |
| Review | Trainer reviews at least 2 substantive PRs per week. |
| Merge | Merge only after CI is green and checklist is complete. |

Branch names MUST be lowercase and descriptive.

| Work type | Example branch |
|---|---|
| Feature | `feat/silver-validation` |
| Fix | `fix/restart-upsert-duplicates` |
| Test | `test/aqi-boundaries` |
| Docs | `docs/runbook-replay-offsets` |
| Chore | `chore/docker-healthchecks` |

## Conventional Commits

Commit messages MUST follow Conventional Commits.

| Type | Example |
|---|---|
| `feat` | feat(silver): quarantine malformed readings |
| `fix` | fix(alerts): keep cooldown per city and category |
| `test` | test(aqi): cover CPCB breakpoint boundaries |
| `docs` | docs(runbook): add bronze to silver replay steps |
| `chore` | chore(deps): update uv lockfile |
| `ci` | ci: add trivy image scan gate |

## Pull request checklist

- Linked GitHub Issue and related FR IDs are listed.
- Lite profile starts from a clean clone.
- New or changed behavior has tests.
- AQI rule changes include boundary tests.
- Coverage stays at line 80% and branch 70% or higher.
- Ruff, mypy, pytest, Gitleaks, pip-audit, and Trivy pass.
- Kafka topics, Delta paths, and PostgreSQL keys remain consistent with the spec.
- Screenshots or demo notes show Grafana changes.
- PR description states any significant AI assistance.
- No secrets, real personal data, or generated bulky files are committed.

## Branch protection

| Protection | Required setting |
|---|---|
| Required checks | lint, type check, tests, coverage, build, security scan. |
| Pull request review | At least one review before merge. |
| Direct pushes | Disabled for `main`. |
| Force pushes | Disabled for `main`. |
| Status freshness | Branch must be up to date before merge. |
| Secret scan | Failing Gitleaks blocks merge. |

## Quality tools and rules

| Tool | Rule |
|---|---|
| Ruff | Lint and format simulator, Spark job, and test modules. |
| mypy | Check application packages with `disallow_untyped_defs` and `no_implicit_optional`. |
| pytest | Run unit, integration, and deterministic streaming tests. |
| pytest-cov | Enforce line 80% and branch 70% gates. |
| AQI tests | AQI module MUST have 100% branch coverage. |
| chispa | Compare Spark DataFrames in transformation tests. |
| pre-commit | Run Ruff, mypy, whitespace, end-of-file, and Gitleaks hooks. |
| pip-audit | Fail on high or critical Python dependency vulnerabilities. |
| Trivy | Fail on high or critical image vulnerabilities unless documented false positive. |
| Docker health checks | Fail local smoke check when Kafka, PostgreSQL, Spark, or Grafana is unhealthy. |

## CI pipeline

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Lint and format | PR and push to `main` | Install uv environment and run Ruff checks. | Any lint or format violation fails. |
| Type check | PR and push to `main` | Run mypy on application packages. | Any type error fails. |
| Unit tests | PR and push to `main` | Run fast tests for AQI, validation, and alert rules. | Failed test or low AQI branch coverage fails. |
| Integration tests | PR and push to `main` | Run selected Spark batch and Kafka contract tests. | Failed transformation, schema, or topic test fails. |
| Coverage | PR and push to `main` | Produce line and branch reports. | Line below 80% or branch below 70% fails. |
| Build | PR and push to `main` | Build pinned local images. | Unpinned image or failed build fails. |
| Security | PR and push to `main` | Run Gitleaks, pip-audit, and Trivy. | Secret or unaccepted high risk fails. |
| Smoke | Push to `main` | Start lite profile and run health checks. | Any Must service unhealthy fails. |

## Docker and Compose requirements

| Requirement | Detail |
|---|---|
| One command | One documented command MUST start lite profile. |
| Profiles | Compose MUST separate lite and standard services. |
| Image tags | All images MUST pin version tags, never `latest`. |
| Kafka | Use one Kafka 4.3.x KRaft broker in Must scope. |
| Spark | Use Java 17 and Spark container limit 2304 MiB with 1500m driver heap in lite mode. |
| PostgreSQL | Persist local data in a named volume. |
| Grafana | Provision PostgreSQL data source and VayuStream dashboard. |
| Health checks | Kafka topics, PostgreSQL readiness, Spark query status, alert poller heartbeat, and Grafana readiness are checked. |
| Ports | Bind development ports to localhost where possible. |

## Environment variables

| Name | Example value | Purpose | Secret |
|---|---|---|---|
| `VAYU_PROFILE` | `lite` | Select lite, standard, or real-data profile. | No |
| `KAFKA_BOOTSTRAP_SERVERS` | `localhost:9092` | Kafka client connection string. | No |
| `RAW_TOPIC` | `vayu.readings.raw` | Raw readings topic. | No |
| `DLQ_TOPIC` | `vayu.readings.dlq` | Invalid readings topic. | No |
| `ALERT_TOPIC` | `vayu.alerts` | Alert event topic. | No |
| `SIMULATOR_SEED` | `vayu-lite-001` | Deterministic fault seed. | No |
| `SIMULATOR_SENSOR_COUNT` | `10` | Number of active sensors. | No |
| `SIMULATOR_INTERVAL_SECONDS` | `5` | Publish interval per sensor. | No |
| `SPARK_DRIVER_MEMORY` | `1500m` | Lite Spark memory limit. | No |
| `DELTA_BASE_PATH` | `./data/delta` | Local Delta table root. | No |
| `CHECKPOINT_BASE_PATH` | `./data/checkpoints` | Streaming checkpoint root. | No |
| `POSTGRES_HOST` | `localhost` | PostgreSQL host. | No |
| `POSTGRES_DB` | `vayustream` | Database name. | No |
| `POSTGRES_USER` | `vayu_app` | Application user. | No |
| `POSTGRES_PASSWORD` | `change-me-local` | Local password from `.env`. | Yes |
| `GRAFANA_ADMIN_USER` | `admin` | Local Grafana login. | No |
| `GRAFANA_ADMIN_PASSWORD` | `change-me-local` | Local Grafana password. | Yes |
| `OPEN_METEO_ENABLED` | `false` | Enable hourly Open-Meteo polling. | No |
| `MAILPIT_SMTP_HOST` | `mailpit` | Optional local mail host. | No |

The student repository MUST include `.env.example` with safe placeholder values. It MUST NOT commit a real `.env` file.

## Versioning and release

| Item | Rule |
|---|---|
| Application version | Use SemVer, starting at `0.1.0`. |
| Tags | Tag final submission as `v1.0.0`. |
| CHANGELOG | Record user-visible pipeline, dashboard, and runbook changes. |
| ADRs | Record stack alternatives, delivery choices, and storage choices. |
| Demo release | Use a GitHub Release for the final demo tag. |

## Dependency management

- Python dependencies MUST be declared in `pyproject.toml` and locked in `uv.lock`.
- Docker image tags MUST be pinned to cohort minor lines.
- Dependency updates MUST run tests and security scans before merge.
- Dependabot MAY open PRs for Python and GitHub Actions dependencies.
- The student MUST review Spark, Delta, Scala, and Kafka connector compatibility together.
- Optional Open-Meteo implementation MUST handle API terms and rate limits before use.

## Definition of Done

A VayuStream task is done only when all applicable items are true.

| Area | Done means |
|---|---|
| Requirement | Linked FR, BR, NFR, or test ID is named in the PR. |
| Behavior | Lite profile behavior matches the specification. |
| Tests | Positive, negative, boundary, or recovery tests are updated. |
| Data quality | Invalid, duplicate, and late records have defined outcomes. |
| Reliability | Replay does not create duplicate Delta or PostgreSQL business keys. |
| Observability | Logs include query name, batch ID, counts, and errors. |
| Documentation | README, runbook, ADR, or dashboard notes are updated. |
| CI | All required checks pass on the PR branch. |
| Review | PR checklist is complete and no unresolved review comments remain. |

[Back to README](../README.md)
