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
- Lite starts offline after initial downloads through doc 06's single entry point; local acceptance remains green.
- New or changed behavior has tests.
- AQI rule changes include boundary tests.
- Coverage stays at line 80% and branch 70% or higher.
- Ruff, mypy, pytest, Gitleaks, pip-audit, and Trivy pass.
- Kafka topics, Delta paths, and PostgreSQL keys remain consistent with the spec.
- Exact JSON/text/CSV snapshots prove inspection changes; no frontend or cloud-deployment task is added.
- PR description states any significant AI assistance.
- No secrets, real personal data, or generated bulky files are committed.

## Branch protection

| Protection | Required setting |
|---|---|
| Required checks | lint, type check, tests, coverage, build, inspection contracts, local acceptance, security scan. |
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
| pytest | Retain at least 45 meaningful unit, integration, deterministic streaming, inspection, and local acceptance tests. |
| pytest-cov | Enforce line 80% and branch 70% gates separately, including Python inspection and ops code. No frontend coverage gate. |
| AQI tests | AQI module MUST have 100% branch coverage. |
| chispa | Compare Spark DataFrames in transformation tests. |
| pre-commit | Run Ruff, mypy, whitespace, end-of-file, and Gitleaks hooks. |
| pip-audit | Fail on high or critical Python dependency vulnerabilities. |
| Trivy | Fail on high or critical image vulnerabilities unless documented false positive. |
| Docker health checks | Fail local smoke check when Kafka/topics, PostgreSQL, the one shared Spark driver's queries/checkpoints, or poller heartbeat is unhealthy. |

## CI pipeline

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Lint and format | PR and push to `main` | Install uv environment and run Ruff checks. | Any lint or format violation fails. |
| Type check | PR and push to `main` | Run mypy on application packages. | Any type error fails. |
| Test inventory | PR and push to `main` | Collect cases and match implemented tests or named reviews to the catalog. | Fewer than 45 meaningful cases or missing Must verification fails; repeated parameter labels alone do not meet the minimum. |
| Unit tests | PR and push to `main` | Run fast tests for AQI, validation, and alert rules. | Failed test or low AQI branch coverage fails. |
| Integration and streaming tests | PR and push to `main` | Run Spark batch tests, close their session, then start the one-driver lite stack for deterministic counts, keys, and recovery/outbox tests. | Failed transformation, schema, topic, B1/B2/B3 count, or replay test fails. Never run two Spark drivers concurrently in lite. |
| Inspection contracts | PR and push to `main` | Seed isolated inspection tables and snapshot all four commands in JSON/text/CSV; test filters, null/empty data, permissions, and atomic files. | TC-E2E-004 and TC-CLI-001 through TC-CLI-006 must pass. Fixtures are independent of the 100-message stream. |
| Local acceptance | PR and push to `main` | After locked/image/JAR preparation, run doc 06 start/demo/stop/start with external runtime network blocked and project-scoped isolated storage. | TC-LOCAL-001 through TC-LOCAL-004 must pass, including exact offline counts, persisted state, and clear failures. |
| Coverage | PR and push to `main` | Produce line and branch reports. | Line below 80% or branch below 70% fails. |
| Build | PR and push to `main` | Build pinned local images. | Unpinned image or failed build fails. |
| Security | PR and push to `main` | Run Gitleaks, pip-audit, and Trivy. | Secret or unaccepted high risk fails. |
| Smoke | PR and push to `main` | Use doc 06 entry points and health CLI; verify fixed loopback listeners and no UI services. | Any Must service unhealthy, extra driver, or published non-loopback port fails. |

Internet is allowed for initial dependencies, image/JAR caching, hosted CI, artifact uploads, and vulnerability database updates. Runtime acceptance uses only simulator/local/recorded fixtures and local service networking. No CI job provisions or tests Kafka/Grafana panels, maps, dashboards, or a student frontend. Performance measurements are recorded after implementation, not claimed by this specification.

## Docker and Compose requirements

| Requirement | Detail |
|---|---|
| One start/stop interface | Implement doc 06's `uv run --no-sync vayustream local start --profile lite` and `uv run --no-sync vayustream local stop`; no alternate startup recipe is required. |
| Profiles | Compose MUST separate lite and standard services. |
| Image tags | All images MUST pin version tags, never `latest`. |
| Kafka | Use one Kafka 4.3.x KRaft broker in Must scope. |
| Spark | Java 17, one shared driver/SparkSession for six queries, proposed aggregate 2048 MiB including Python/non-heap memory and 1280m JVM heap; `local[2]`, shuffle partitions 2, no Spark UI or published driver port. |
| PostgreSQL | Persist gold/alert/poller/outbox data in a project-scoped named volume; initialize migrations and SELECT-only inspector idempotently. |
| Source and seeds | Simulator is default and idle until a requested data operation; fictional stations and named fixture initialize locally. Open-Meteo is explicit opt-in, never required or an automatic substitute. |
| Persistent storage | Retain KRaft cluster ID/data, Delta/checkpoints, demo manifest, and evidence through stop/start. Only confirmed local reset deletes project resources. |
| Health checks | Kafka/topics, PostgreSQL tables, one driver's six query states/isolated checkpoints, and alert-poller heartbeat. No console is required. |
| Ports | Host mappings `127.0.0.1:19092` and `127.0.0.1:15433`; internal Kafka `kafka:29092`; controller/driver ports never published. Optional standard Mailpit uses doc 06's disjoint loopback ports. |

## Environment variables

| Name | Example value | Purpose | Secret |
|---|---|---|---|
| `VAYU_PROFILE` | `lite` | Select lite, standard, or real-data profile. | No |
| `VAYU_SOURCE_MODE` | `simulator` | Explicit default; `open-meteo` is optional real-source mode. | No |
| `KAFKA_BOOTSTRAP_SERVERS` | `127.0.0.1:19092` | Host Kafka client string; container clients use `kafka:29092`. | No |
| `RAW_TOPIC` | `vayu.readings.raw` | Raw readings topic. | No |
| `DLQ_TOPIC` | `vayu.readings.dlq` | Invalid readings topic. | No |
| `ALERT_TOPIC` | `vayu.alerts` | Alert event topic. | No |
| `SIMULATOR_SEED` | `vayu-lite-001` | Deterministic fault seed. | No |
| `SIMULATOR_SENSOR_COUNT` | `10` | Number of active sensors. | No |
| `SIMULATOR_INTERVAL_SECONDS` | `5` | Publish interval per sensor. | No |
| `SIMULATOR_AUTOSTART` | `false` | Keep publisher ready/idle until explicit demo or continuous data operation. | No |
| `SPARK_DRIVER_MEMORY` | `1280m` | Lite JVM heap inside the proposed 2048 MiB all-process Spark budget. | No |
| `DELTA_BASE_PATH` | `./data/delta` | Local Delta table root. | No |
| `CHECKPOINT_BASE_PATH` | `./data/checkpoints` | Streaming checkpoint root. | No |
| `POSTGRES_HOST` | `127.0.0.1` | Host CLI PostgreSQL address; containers use internal service name. | No |
| `POSTGRES_PORT` | `15433` | Host CLI port; containers use internal port `5432`. | No |
| `POSTGRES_DB` | `vayustream` | Database name. | No |
| `POSTGRES_USER` | `vayu_app` | Application user. | No |
| `POSTGRES_PASSWORD` | `change-me-local` | Local password from `.env`. | Yes |
| `POSTGRES_INSPECT_USER` | `vayu_inspector` | SELECT-only serving-table access. | No |
| `POSTGRES_INSPECT_PASSWORD` | `change-me-local` | Local inspector password from `.env`. | Yes |
| `OPEN_METEO_ENABLED` | `false` | Enable hourly Open-Meteo polling. | No |
| `MAILPIT_SMTP_HOST` | `mailpit` | Optional local mail host. | No |

The student repository MUST include `.env.example` with safe placeholder values. It MUST NOT commit a real `.env` file.

## Versioning and release

| Item | Rule |
|---|---|
| Application version | Use SemVer, starting at `0.1.0`. |
| Tags | Tag final submission as `v1.0.0`. |
| CHANGELOG | Record pipeline, inspection CLI, local-operation, and runbook changes. |
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
| Documentation | README, runbook, ADR, or exact inspection/local contracts are updated. |
| CI | All required checks pass on the PR branch. |
| Review | PR checklist is complete and no unresolved review comments remain. |
| Local reliability | Applicable offline demo, persistent stop/start, invalid-input/dependency, and exact inspection cases in doc 09 pass. |

[Back to README](../README.md)
