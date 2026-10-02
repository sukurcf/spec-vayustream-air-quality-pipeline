# Tech stack and setup

Purpose: This document defines the approved VayuStream toolchain, local setup, hardware limits, and learning order.

## Reference stack

| Category | Tool | Reference version | Purpose | Why |
|---|---|---|---|---|
| Language | Python | 3.12.x | Simulator, Spark jobs, inspection/local CLIs, tests | Cohort baseline; CLI parsing/JSON/CSV use the standard library. |
| Package manager | uv | 0.12.x | Manage Python envs and lockfile | Fast installs and reproducible `uv.lock`. |
| Streaming broker | Apache Kafka | 4.3.x; image `apache/kafka:4.3.1` | Raw, DLQ, and alert topics | KRaft mode matches modern Kafka interviews. |
| Kafka client | confluent-kafka | 2.15.x | Producer and optional alert consumer | Mature Python client with idempotent producer settings. |
| Processing | PySpark | 4.2.x | Structured Streaming transformations | Required for Spark DataFrames and micro-batches. |
| Lakehouse | delta-spark | 4.4.x | Bronze, Silver, Quarantine Delta tables | Adds ACID tables and time travel. |
| Kafka connector | spark-sql-kafka | `org.apache.spark:spark-sql-kafka-0-10_2.13:4.2.0` | Spark reads and writes Kafka | Must match Spark 4.2 and Scala 2.13. |
| Runtime | Java | 17 | Spark runtime | Spark 4 supports Java 17 or 21; 17 is stable. |
| Database | PostgreSQL | 18.x; pin `postgres:18.6` or newer 18.x | Gold/alert tables and read-only inspection | Strong SQL and idempotent upserts. |
| Containers | Docker Engine | 29.x | Run local services | Same workflow on WSL2, macOS, and Linux. |
| Compose | Docker Compose | v5.x | One-command local stack | Simple multi-service orchestration. |
| Notification sink | Mailpit | 1.31.x | Optional e-mail testing | Maintained replacement for MailHog. |
| Tests | pytest | 9.1.x | Unit and integration tests | Standard Python test runner. |
| Coverage | coverage / pytest-cov | 7.16.x / 7.1.x | Coverage gates | Enforces 80% line and 70% branch coverage. |
| Spark assertions | chispa | 0.12.x | DataFrame comparisons | Clear equality checks for Spark tests. |
| Integration tests | Testcontainers | 4.15.x | Optional Kafka and PostgreSQL tests | Starts disposable real dependencies. |
| Schema validation | jsonschema or Pydantic | jsonschema current docs; Pydantic 2.13.x | Validate reading events | Keeps event contracts explicit. |
| Quality | Ruff | 0.16.x | Lint and format | Fast formatter and Python lint rules. |
| Types | mypy | 2.4.x | Static type checking | Catches data-contract mistakes early. |
| Hooks | pre-commit | 4.6.x | Local quality gates | Runs checks before commits. |
| Secret scan | Gitleaks | 8.30.x | Prevent committed secrets | Required for NFR-SEC-01. |
| Dependency audit | pip-audit | 2.10.x | Python vulnerability scan | CI security gate. |
| Image scan | Trivy | 0.75.x | Container vulnerability scan | CI security gate. |

### Optional built-in operations consoles, not implementation scope

| Tool | Existing version family | Restriction |
|---|---|---|
| kafbat kafka-ui | 1.5.x | MAY be inspected separately on a 16 GB host for topic operations. Never a business interface, student frontend task, CI gate, or grader requirement. |
| Grafana OSS | 13.x | MAY be inspected separately for operations only. No student panels, maps, custom dashboards, provisioning exercise, or exported UI artifact. |

Neither console is part of either required runtime profile or the lite memory budget. Students may not replace Python inspection evidence with a console screenshot.

## Mandatory tool licence table

Students MUST use tools only for local educational work unless they verify separate terms.

| Tool | Licence or terms | Condition for free local educational use |
|---|---|---|
| Python | Python Software Foundation License | Free for local development and redistribution. |
| uv | MIT or Apache-2.0 | Free for package management in the student repository. |
| Apache Kafka | Apache-2.0 | Free for one local KRaft broker. |
| confluent-kafka Python client | Apache-2.0 | Free for local producer and consumer clients. |
| PySpark and Spark Kafka connector | Apache-2.0 | Free for local Spark jobs and tests. |
| delta-spark | Apache-2.0 | Free for local Delta tables. |
| Java 17 OpenJDK | GPLv2 with Classpath Exception | Free when using an OpenJDK distribution locally. |
| PostgreSQL | PostgreSQL License | Free for local Gold/alert serving tables. |
| Docker Engine and Compose | Apache-2.0 for the engine and Compose | Free for local Linux engine use. |
| Docker Desktop | Docker subscription terms | Free only when the student's use fits Docker's personal, education, or small-business terms. |
| pytest | MIT | Free for local and CI test runs. |
| coverage and pytest-cov | Apache-2.0 and MIT | Free for coverage gates. |
| chispa | MIT | Free for Spark DataFrame assertions. |
| jsonschema or Pydantic | MIT | Free for event validation. |
| Ruff | MIT | Free for linting and formatting. |
| mypy | MIT | Free for static type checking. |
| pre-commit | MIT | Free for local hooks. |
| Gitleaks | MIT | Free for secret scanning. |
| pip-audit | Apache-2.0 | Free for Python dependency audits. |
| Trivy | Apache-2.0 | Free for container and dependency scans. |

## Allowed alternatives

Each alternative requires an ADR before implementation.

| Alternative | Allowed for | ADR question |
|---|---|---|
| Java 21 | Spark runtime | Does Spark 4.2, Delta 4.4, and the local image support it on all laptops? |
| Pydantic instead of jsonschema | Event validation | Does it preserve the JSON Schema contract used in docs? |
| Webhook instead of Mailpit | Alert notification | How are at-least-once retries and duplicate `alert_id` values handled? |
| Open-source schema registry with Avro | Could schema feature | How does it fit 8 GB lite mode and local setup time? |
| Maintained local object storage | Could Delta experiment on standard profile | What licence, image status, and local resource budget prove it fits without cloud deployment? |

## Not allowed

| Tool or practice | Reason |
|---|---|
| Kafka 3.x or ZooKeeper mode | VayuStream teaches Kafka 4.x KRaft. |
| `latest` Docker image tags | Reproducibility and grading require pinned versions. |
| Bitnami images or charts | The free catalog changed in 2025. |
| Promtail | It is deprecated; the specified structured logs provide operations evidence. |
| MailHog | It is unmaintained; Mailpit is the approved local mail tool. |
| Cloud-deployment or paid-account exercises | The project runs locally; these are not optional or bonus work. |
| Exactly-once webhook or e-mail claims | External notifications are at-least-once only. |
| UI services in lite or student-built frontend at any priority | RAM belongs to data services; business evidence is Python JSON/text/CSV. |

## Hardware profiles

| Profile | Laptop target | Services | Proposed limits, not measured results |
|---|---:|---|---|
| Lite | 4 cores, 8 GB RAM, 20 GB free disk | One Kafka broker, one shared local Spark driver, PostgreSQL, simulator/poller, short-lived CLI | 10 stations, 5 cities, 5-second steady interval, `local[2]`, shuffle partitions 2, max offsets per trigger 100; no UI, Mailpit, registry, or object-store services. |
| Standard | 16 GB RAM recommended, 40 GB free disk recommended | Same core services, optional Mailpit | 50 stations, 10 cities, full pollutants, opt-in Open-Meteo; no required UI services. Extra Could storage/schema services need a separate ADR budget. |

All six Bronze/Silver/Gold queries MUST share one driver and one SparkSession. The proposed Spark cgroup budget includes JVM heap, native/non-heap memory, and Python workers, not six separate per-query heaps. Disable Spark's built-in UI in lite mode; do not publish driver ports.

| Service/process group | Proposed lite limit | Proposed standard limit |
|---|---:|---:|
| Kafka, one KRaft broker | 768 MiB including 512 MiB heap | 1024 MiB including 768 MiB heap |
| Shared Spark driver plus Python workers, all six queries | 2048 MiB including 1280m JVM heap | 4096 MiB including 2560m JVM heap |
| PostgreSQL | 256 MiB; shared buffers at most 64 MiB, max connections 20 | 512 MiB |
| Simulator and alert poller combined | 192 MiB | 256 MiB |
| Short-lived inspection/local CLI | 128 MiB | 128 MiB |
| Optional Mailpit | Not started | 128 MiB |
| Total at stated concurrency | 3392 MiB | 6144 MiB including Mailpit |

Lite leaves about 704 MiB in a 4 GB WSL/VM allocation for Linux/container overhead; the host retains the other 4 GB. These are proposed planning budgets, not a claimed measured fit. Trainer pre-check MUST record actual peak RSS/cgroup use, swap, and batch latency. Reduce batching/concurrency if necessary without changing 10 stations, watermark rules, or Must behavior. Run batch-Spark tests and the streaming stack sequentially so a second driver does not consume the lite budget.

Windows users MUST use WSL2 Ubuntu 24.04. Use these `.wslconfig` values unless an ADR gives a written reason.

| Host RAM | WSL memory | WSL swap |
|---|---|---|
| 8 GB | 4 GB | 4 GB |
| 16 GB | 8 GB | 4 GB |

## Local setup checklist

### Windows 11 with WSL2 Ubuntu 24.04

1. Enable WSL2 and install Ubuntu 24.04.
2. Set `.wslconfig` from the hardware table.
3. Install Docker Desktop with WSL2 integration enabled.
4. Install Git, VS Code, and the Remote - WSL extension.
5. Install Python 3.12 and uv inside Ubuntu.
6. Install Java 17 inside Ubuntu.
7. Clone the student repository `vayustream` inside the WSL filesystem.
8. Complete initial locked dependency/image/JAR downloads and create `.env` from `.env.example`.
9. Use the single lite start entry point below; check Kafka, shared Spark, PostgreSQL, and poller health.
10. Run lint, type checks, tests, and coverage before the first PR.

### macOS

1. Install Docker Desktop or a compatible Docker engine.
2. Install Git, Python 3.12, uv, and Java 17.
3. Install VS Code and Python, Docker, and YAML extensions.
4. Clone `vayustream` and create `.env` from `.env.example`.
5. Start lite profile and run the smoke checks.

### Linux

1. Install Docker Engine 29.x and Compose v5.x.
2. Add the user to the Docker group only if local policy allows it.
3. Install Git, Python 3.12, uv, and Java 17.
4. Clone `vayustream` and create `.env` from `.env.example`.
5. Start lite profile and run the smoke checks.

## Local operation contract students MUST implement

This repository supplies specifications only. The following are commands/interfaces for the student implementation, not an application that has been run.

Initial `uv sync --frozen`, pinned image pulls/builds, Java/tool downloads, and Spark Kafka/Delta JAR downloads require internet. Bundle or pre-cache the approved connector/Delta artifacts so Spark does not resolve Maven packages during offline runtime. GitHub submission and hosted CI require internet. After these downloads, startup, simulator, tests, demonstration, and inspection MUST run without external network access. No paid account, public hostname, API key, cloud resource, or GitHub connection is needed locally.

### Single start and stop entry points

| Command from the student repository | Contract |
|---|---|
| Start: `uv run --no-sync vayustream local start --profile lite` | The one default start entry point. Preflight Docker, disk, ports, and configuration; initialize/reuse storage, Kafka/topics, PostgreSQL, one shared Spark driver, and poller; wait for health within 5 minutes after downloads. Source mode is `simulator`. Exit `0` only when healthy, and print the exact healthy JSON below. |
| Stop: `uv run --no-sync vayustream local stop` | The one stop entry point. Flush producer within 10 seconds, finish/checkpoint active Spark work or retain uncommitted work for replay, commit poller state, and stop project services. Output `{"status":"stopped","data_preserved":true}`; never delete storage. |
| Health: `uv run --no-sync vayustream local health --format json` | Verify broker/topics, six named queries in one driver, checkpoint isolation, PostgreSQL tables, and poller heartbeat; no remote request or console is involved. |
| Demonstrate: `uv run --no-sync vayustream demo --fixture vayu-lite-001` | Run the exact B1/B2/B3 simulator fixture using explicit progress/watermark barriers in the existing shared driver; return the exact summary below. |
| Confirmed reset: `uv run --no-sync vayustream local reset --confirm-delete-data` | Stop services and delete only this project's generated local data/volumes. Refuse without this exact confirmation flag; never delete committed fixtures, source code, or another project's data. |

Initialization is idempotent: retain the KRaft cluster ID and broker volume, create only missing topics with doc 08 retention/partition settings, apply versioned PostgreSQL gold/alert/poller/outbox migrations, and create writer and SELECT-only `vayu_inspector` users. Generate/load the fictional `stations_lite.csv` with exactly 10 active stations in 5 cities and local seed `vayu-lite-001`. Initialize isolated Delta/checkpoint paths and a persisted demonstration manifest. Do not reformat Kafka, recreate populated schemas, or overwrite existing checkpoints on restart.

The start entry point launches the data services and makes the simulator ready, but leaves publishing idle until the explicit seeded demonstration or a requested continuous simulator operation. This prevents uncontrolled messages from contaminating the 100-event fixture. `vayustream simulate --duration-seconds 900` is a data operation for the steady throughput trial, not a second stack-start mechanism. Standard uses the same start entry point with `--profile standard`.

### Fixed loopback ports

| Service | Host binding | Internal listeners | Health evidence |
|---|---|---|---|
| Kafka client listener | `127.0.0.1:19092` | Host listener 9092; bridge broker listener 29092; controller 9093 | Broker metadata and all three topic configurations are readable. |
| PostgreSQL | `127.0.0.1:15433` | 5432 | `SELECT 1`, migration version, and gold/alert/poller/outbox tables exist. |
| Optional standard Mailpit SMTP/API | `127.0.0.1:11026`, `127.0.0.1:18026` | 1025, 8025 | Local notification capture API; console inspection optional, never business evidence. |

Host Kafka clients use `127.0.0.1:19092`; container clients use `kafka:29092`. Configure matching advertised listeners so host metadata does not redirect to an unreachable container name. Never publish controller/bridge broker/Spark driver ports. Lite disables Spark's built-in UI and starts no Kafka/Grafana UI service. Required ports are disjoint from ShopSight's `15434/18080/11027/18027`. Publish only on `127.0.0.1`; an occupied fixed port fails with `local-port-in-use`, exit `3`, not an automatic port change.

Healthy default-lite JSON is exactly:

```json
{"status":"healthy","services":{"kafka":"healthy","spark":"healthy","postgres":"healthy","alert_poller":"healthy"},"source_mode":"simulator","active_stations":10,"spark_driver_count":1,"query_count":6}
```

Successful start and the health command MUST each print this single JSON object to stdout. Send progress messages to stderr. On startup failure, print the documented error to stderr and exit nonzero. Do not print a successful health object.

### Deterministic offline demonstration

Use committed station, AQI, and 100-message fixtures with fixed timestamps, seed, and B1/B2/B3 boundaries from [doc 08](08-pipeline-specification.md). The demo requires empty fixture sinks or the same completed demonstration manifest. On repeat it verifies existing results rather than publishing another 100 messages. A non-fixture/populated workspace fails with `demo-workspace-not-clean`; it never silently deletes or replaces data. Startup never republishes a completed seed.

Persist operator drop counters and completed offsets in demonstration evidence/manifest, keyed by logical query name and batch ID. Duplicate-drop counts come from Spark state-operator progress as required, not inferred from row subtraction. Replayed progress does not add a batch twice; stop/start retains the original `5` duplicate and `2` late drops for repeat verification.

Demo output is exactly:

```json
{"fixture":"vayu-lite-001","bronze_rows":100,"silver_rows":87,"dlq_messages":6,"quarantine_rows":6,"duplicate_events_dropped":5,"late_events_dropped":2,"final_station_windows":10,"final_city_windows":5,"station_windows":11,"city_windows":6,"alert_events":0}
```

The seed has only one FINAL window per city, so it cannot satisfy the two-adjacent-FINAL alert threshold; `alert_events=0` is deliberate. Separate fixed FINAL-window/outbox fixtures prove trigger, cooldown, duplicate publication, and recovery at AQI `200`; do not add those rows to the 100-event demo or change its counts.

Inspect through `uv run --no-sync vayustream inspect current --format json`, `inspect windows --limit 100 --format csv`, `inspect completeness --level station --limit 100 --format text`, and `inspect alerts --format json` using the same command prefix. Exact columns, order, null/empty behavior, filters, and independent inspection-contract fixtures are in doc 08. These commands produce 5 current-city rows, 6 stored city-window rows, 11 station-completeness rows, and an empty alert result for the demo.

### Sources, persistence, and failure checks

Default `VAYU_SOURCE_MODE=simulator`, `OPEN_METEO_ENABLED=false`. Open-Meteo is separately opt-in through `--source-mode open-meteo`/explicit configuration and is unnecessary for local acceptance. Optional `real-data` profile uses standard hardware and event `profile=real-data`; the same start entry point applies. Health reports configured source and actual station counts in non-default profiles. Tests use recorded API responses and do not poll externally. An Open-Meteo failure logs `open-meteo-unavailable`; it does not silently enable or substitute the simulator.

Persist project-scoped volumes `vayustream_kafka_data`, `vayustream_pg_data`, `vayustream_delta_data`, and `vayustream_checkpoints`, plus bind mounts for the demo manifest, status/heartbeat files, and output evidence. Stop/start retains offsets, Delta/checkpoints, gold keys/status, poller cursor, streak/cooldown, and outbox rows. Ignore generated data in Git. Only explicitly confirmed reset deletes these local resources.

Invalid profile/filter/station, unavailable Docker/Kafka/PostgreSQL, port collision, invalid lite services/driver count, and unconfirmed reset return stable stderr errors and nonzero exit codes from doc 08. No partial inspection output or automatic destructive recovery is allowed. Trainer pre-check, CI, and grading MUST cover `TC-LOCAL-001` through `TC-LOCAL-004` in doc 09.

## Trainer pre-check

The trainer MUST complete this checklist before week 1.

| Check | Expected result |
|---|---|
| Start lite profile on an 8 GB/four-core laptop | All Must services become healthy with one broker, one shared driver, and no UI services; TC-LOCAL-001 passes. |
| Run simulator for 15 minutes | 10 stations publish about 1,800 readings. |
| Verify proposed memory budget | Measure all-process peak RAM/swap, including Spark Python/non-heap memory, against the 3392 MiB proposal; do not assume it fits. |
| Verify inspection CLI | JSON/text/CSV commands return exact doc 08 contracts, window status, indicative basis, and null AQI labels. |
| Verify deterministic test data | 100-message test contains the registered fault counts. |
| Verify shutdown | Producer exits within 10 seconds after SIGTERM. |
| Verify ports and persistence | Fixed ports bind to `127.0.0.1`; TC-LOCAL-003/004 prove persisted restart and clear failures. |

## Free accounts needed

| Account | Required | Purpose |
|---|---|---|
| GitHub | Yes | Student repository, issues, pull requests, Actions. |
| Docker Hub | Recommended | Avoid image pull limits during setup. |
| Open-Meteo API key | No | API is free and keyless for non-commercial use. |
| Paid/cloud account | No | No cloud-deployment exercise is allowed at any priority. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---:|---|---:|---|
| 1 | Linux, Git, uv, Docker basics | 5 | Student can start and inspect local services. |
| 2 | Kafka topics, partitions, offsets, keys, retention | 6 | Student can publish, consume, and replay raw events. |
| 3 | Spark DataFrames and lazy evaluation | 5 | Student can explain transformations, actions, partitions, and shuffles. |
| 4 | Structured Streaming micro-batches | 6 | Student can explain triggers, checkpoints, watermarks, and output modes. |
| 5 | Delta Lake medallion architecture | 4 | Student can explain Bronze, Silver, Gold, ACID, and time travel. |
| 6 | CPCB AQI rules and data quality | 4 | Student can implement boundary tests and quarantine reasons. |
| 7 | PostgreSQL, Python inspection, and local reliability | 4 | Student explains idempotent upserts, exact CLI output, persisted restart, and at-least-once alerts. |
| 8 | Testing and CI | 6 | Student can meet coverage and CI gates. |

Week 1 SHOULD spend about 30 hours on learning and setup. Later weeks reinforce these topics through implementation.

## Reference documentation links

| Tool | Official link verified for this spec |
|---|---|
| Python | https://www.python.org/doc/ |
| uv | https://docs.astral.sh/uv/ |
| Ruff | https://docs.astral.sh/ruff/ |
| mypy | https://mypy.readthedocs.io/ |
| pytest | https://docs.pytest.org/ |
| coverage | https://coverage.readthedocs.io/ |
| pre-commit | https://pre-commit.com/ |
| pip-audit | https://github.com/pypa/pip-audit |
| Docker Engine | https://docs.docker.com/engine/ |
| Docker Compose | https://docs.docker.com/compose/ |
| PostgreSQL | https://www.postgresql.org/docs/ |
| Apache Kafka | https://kafka.apache.org/documentation/ |
| confluent-kafka | https://docs.confluent.io/kafka-clients/python/current/overview.html |
| PySpark | https://spark.apache.org/docs/latest/api/python/ |
| Delta Lake | https://docs.delta.io/latest/index.html |
| Spark Kafka connector | https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html |
| Java 17 | https://docs.oracle.com/en/java/javase/17/ |
| Grafana OSS, optional built-in operations inspection only | https://grafana.com/docs/grafana/latest/ |
| Mailpit | https://mailpit.axllent.org/ |
| Trivy | https://trivy.dev/latest/docs/ |
| Gitleaks | https://gitleaks.io/ |
| chispa | https://github.com/MrPowers/chispa |
| Testcontainers | https://testcontainers.com/guides/getting-started-with-testcontainers-for-python/ |
| jsonschema | https://python-jsonschema.readthedocs.io/ |
| Pydantic | https://docs.pydantic.dev/ |
| kafbat kafka-ui, optional built-in operations inspection only | https://ui.docs.kafbat.io/ |
| Open-Meteo Air Quality API | https://open-meteo.com/en/docs/air-quality-api |

[Back to README](../README.md)
