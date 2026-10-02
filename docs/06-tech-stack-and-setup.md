# Tech stack and setup

Purpose: This document defines the approved VayuStream toolchain, local setup, hardware limits, and learning order.

## Reference stack

| Category | Tool | Reference version | Purpose | Why |
|---|---|---|---|---|
| Language | Python | 3.12.x | Simulator, Spark jobs, tests, tooling | Cohort baseline with broad data-tool support. |
| Package manager | uv | 0.12.x | Manage Python envs and lockfile | Fast installs and reproducible `uv.lock`. |
| Streaming broker | Apache Kafka | 4.3.x; image `apache/kafka:4.3.1` | Raw, DLQ, and alert topics | KRaft mode matches modern Kafka interviews. |
| Kafka client | confluent-kafka | 2.15.x | Producer and optional alert consumer | Mature Python client with idempotent producer settings. |
| Processing | PySpark | 4.2.x | Structured Streaming transformations | Required for Spark DataFrames and micro-batches. |
| Lakehouse | delta-spark | 4.4.x | Bronze, Silver, Quarantine Delta tables | Adds ACID tables and time travel. |
| Kafka connector | spark-sql-kafka | `org.apache.spark:spark-sql-kafka-0-10_2.13:4.2.0` | Spark reads and writes Kafka | Must match Spark 4.2 and Scala 2.13. |
| Runtime | Java | 17 | Spark runtime | Spark 4 supports Java 17 or 21; 17 is stable. |
| Database | PostgreSQL | 18.x; pin `postgres:18.6` or newer 18.x | Gold tables and Grafana source | Strong SQL and idempotent upserts. |
| Dashboard | Grafana OSS | 13.x | AQI, trend, and alert panels | Common observability tool with PostgreSQL support. |
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
| Kafka UI | kafbat kafka-ui | 1.5.x | Optional topic inspection | Useful in standard profile only. |

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
| PostgreSQL | PostgreSQL License | Free for the local Gold and Grafana database. |
| Grafana OSS | AGPLv3 | Free for local use; keep source and licence notices. |
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
| TypeScript Kafka simulator | Simulator | Why is Python not enough for the student's learning goals? |
| Open-source schema registry with Avro | Could schema feature | How does it fit 8 GB lite mode and local setup time? |
| Maintained object storage | Could Delta experiment | What current licence and image status proves it is safe to use? |

## Not allowed

| Tool or practice | Reason |
|---|---|
| Kafka 3.x or ZooKeeper mode | VayuStream teaches Kafka 4.x KRaft. |
| `latest` Docker image tags | Reproducibility and grading require pinned versions. |
| Bitnami images or charts | The free catalog changed in 2025. |
| Promtail | It is deprecated; use Grafana Alloy only in optional observability work. |
| MailHog | It is unmaintained; Mailpit is the approved local mail tool. |
| Managed cloud Kafka as Must scope | The project MUST run locally and free. |
| Exactly-once webhook or e-mail claims | External notifications are at-least-once only. |
| Kafka UI in lite profile | It consumes RAM needed by Spark and PostgreSQL. |

## Hardware profiles

| Profile | Laptop RAM | Services | Exact limits |
|---|---:|---|---|
| Lite | 8 GB | 1 Kafka broker, Spark local driver, PostgreSQL, Grafana, simulator | 10 sensors, 5 cities, 5-second interval, Kafka 768 MiB with 512 MiB heap, Spark 2304 MiB with 1500m heap, PostgreSQL 384 MiB, Grafana 192 MiB, simulator 128 MiB, no Kafka UI, no Mailpit by default. |
| Standard | 16 GB | Lite services plus Kafka UI and optional Mailpit | 50 sensors, 10 cities, full pollutant set, optional Open-Meteo polling, standard dashboard. |

Windows users MUST use WSL2 Ubuntu 24.04. Use these `.wslconfig` values unless an ADR gives a written reason.

| Laptop | `.wslconfig` memory | `.wslconfig` swap |
|---|---:|---:|
| 8 GB laptop | `memory=4GB` | `swap=4GB` |
| 16 GB laptop | `memory=8GB` | `swap=4GB` |

## Local setup checklist

### Windows 11 with WSL2 Ubuntu 24.04

1. Enable WSL2 and install Ubuntu 24.04.
2. Set `.wslconfig` from the hardware table.
3. Install Docker Desktop with WSL2 integration enabled.
4. Install Git, VS Code, and the Remote - WSL extension.
5. Install Python 3.12 and uv inside Ubuntu.
6. Install Java 17 inside Ubuntu.
7. Clone the student repository `vayustream` inside the WSL filesystem.
8. Run the project setup target and create `.env` from `.env.example`.
9. Start lite profile and check Kafka, Spark, PostgreSQL, and Grafana health.
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

## Trainer pre-check

The trainer MUST complete this checklist before week 1.

| Check | Expected result |
|---|---|
| Start lite profile on an 8 GB laptop | All Must services become healthy without Kafka UI. |
| Run simulator for 15 minutes | 10 stations publish about 1,800 readings. |
| Verify Spark memory | Spark container stays within 2304 MiB and driver heap is 1500m. |
| Verify dashboard | Grafana shows current AQI, trend, and alerts panels. |
| Verify deterministic test data | 100-message test contains the registered fault counts. |
| Verify shutdown | Producer exits within 10 seconds after SIGTERM. |
| Verify ports | Development ports bind only to localhost. |

## Free accounts needed

| Account | Required | Purpose |
|---|---|---|
| GitHub | Yes | Student repository, issues, pull requests, Actions. |
| Docker Hub | Recommended | Avoid image pull limits during setup. |
| Grafana Cloud | No | Not needed; Grafana OSS runs locally. |
| Open-Meteo API key | No | API is free and keyless for non-commercial use. |
| Cloud provider | No | Cloud is out of Must scope. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---:|---|---:|---|
| 1 | Linux, Git, uv, Docker basics | 5 | Student can start and inspect local services. |
| 2 | Kafka topics, partitions, offsets, keys, retention | 6 | Student can publish, consume, and replay raw events. |
| 3 | Spark DataFrames and lazy evaluation | 5 | Student can explain transformations, actions, partitions, and shuffles. |
| 4 | Structured Streaming micro-batches | 6 | Student can explain triggers, checkpoints, watermarks, and output modes. |
| 5 | Delta Lake medallion architecture | 4 | Student can explain Bronze, Silver, Gold, ACID, and time travel. |
| 6 | CPCB AQI rules and data quality | 4 | Student can implement boundary tests and quarantine reasons. |
| 7 | PostgreSQL, Grafana, and alert semantics | 4 | Student can explain idempotent upserts and at-least-once alerts. |
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
| Grafana OSS | https://grafana.com/docs/grafana/latest/ |
| Mailpit | https://mailpit.axllent.org/ |
| Trivy | https://trivy.dev/latest/docs/ |
| Gitleaks | https://gitleaks.io/ |
| chispa | https://github.com/MrPowers/chispa |
| Testcontainers | https://testcontainers.com/guides/getting-started-with-testcontainers-for-python/ |
| jsonschema | https://python-jsonschema.readthedocs.io/ |
| Pydantic | https://docs.pydantic.dev/ |
| kafbat kafka-ui | https://ui.docs.kafbat.io/ |
| Open-Meteo Air Quality API | https://open-meteo.com/en/docs/air-quality-api |

[Back to README](../README.md)
