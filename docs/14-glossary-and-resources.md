# Glossary and resources

Purpose: This document defines VayuStream terms and lists verified official documentation and free learning resources.

## Glossary

| Term | Simple definition |
|---|---|
| AQI | Air Quality Index, a number and category that summarize pollutant concentration. |
| CPCB | Central Pollution Control Board, the Indian body whose AQI method is the reference. |
| Sub-index | AQI score for one pollutant before taking the maximum value. |
| Breakpoint | Concentration range and AQI range used for linear interpolation. |
| Indicative AQI | VayuStream's 15-minute AQI signal, not the official daily AQI. |
| Pollutant | Measured air contaminant such as PM2.5, PM10, or NO2. |
| PM2.5 | Fine particulate matter with diameter 2.5 micrometres or less. |
| PM10 | Particulate matter with diameter 10 micrometres or less. |
| NO2 | Nitrogen dioxide, a traffic and combustion pollutant. |
| CO | Carbon monoxide, measured in mg/m³ in VayuStream. |
| O3 | Ozone, used in standard and daily AQI work. |
| NH3 | Ammonia, an optional standard-profile pollutant. |
| Kafka topic | Named Kafka stream that stores ordered records in partitions. |
| Partition | Ordered slice of a Kafka topic. |
| Offset | Record position inside one Kafka partition. |
| KRaft | Kafka metadata mode that removes ZooKeeper. |
| Producer idempotence | Kafka producer feature that reduces duplicates during retries. |
| At-least-once | Delivery may repeat, so consumers must deduplicate by stable IDs. |
| Stable ID | Identifier such as `event_id` or `alert_id` that stays the same across retries. |
| Structured Streaming | Spark engine that processes streams as repeated micro-batches. |
| Micro-batch | Small batch of newly available records processed by Spark. |
| Watermark | Event-time limit that controls late data state and late drops. |
| Tumbling window | Fixed non-overlapping time window, such as 15 minutes. |
| Checkpoint | Stored Spark progress and state used for restart safety. |
| Delta Lake | Table format with ACID transactions on data files. |
| Bronze | Raw audit layer that keeps Kafka payloads and metadata. |
| Silver | Valid, typed, deduplicated, enriched reading layer. |
| Gold | Aggregated analytics layer for dashboard and alerts. |
| Quarantine | Storage for invalid events with rejection reasons. |
| DLQ | Dead-letter queue topic for invalid events. |
| Stream-static join | Join between streaming readings and static station reference data. |
| foreachBatch | Spark hook that writes each micro-batch to an external sink. |
| Idempotent upsert | Write that can run again and still leaves one row per business key. |
| Completeness | Unique valid readings divided by expected readings times 100. |
| Grafana | Dashboard tool that reads PostgreSQL gold tables. |
| PostgreSQL | Relational database used for dashboard-friendly gold and alert tables. |
| Open-Meteo | Optional free API source for external air-quality readings. |
| Retention | Time Kafka keeps records before deletion. |
| Cooldown | Time window that prevents repeated alerts for the same city and category. |
| Recovery alert | Alert sent when unhealthy AQI returns to Moderate or better. |
| Consumer lag | Difference between latest Kafka offset and consumed offset. |
| Replay | Reprocessing stored Kafka or Bronze data after a chosen point. |

## Official documentation links

| Tool | Official documentation |
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

## Free learning resources

| Topic | Resource |
|---|---|
| Python | Official Python tutorial: https://docs.python.org/3/tutorial/ |
| uv | uv guides: https://docs.astral.sh/uv/guides/ |
| Git | Git book: https://git-scm.com/book/en/v2 |
| Docker | Docker getting started: https://docs.docker.com/get-started/ |
| Kafka | Apache Kafka quickstart and design docs: https://kafka.apache.org/documentation/ |
| Spark | Spark SQL and DataFrame guide: https://spark.apache.org/docs/latest/sql-programming-guide.html |
| Structured Streaming | Spark Structured Streaming guide: https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html |
| Delta Lake | Delta Lake getting started: https://docs.delta.io/latest/quick-start.html |
| PostgreSQL | PostgreSQL tutorial: https://www.postgresql.org/docs/current/tutorial.html |
| Grafana | Grafana fundamentals: https://grafana.com/docs/grafana/latest/fundamentals/ |
| AQI | CPCB official AQI resources from the Central Pollution Control Board website. |
| Testing | pytest getting started: https://docs.pytest.org/en/stable/getting-started.html |

## Suggested reading order

1. Read the README and project overview first.
2. Read users, functional requirements, and non-functional requirements.
3. Study Kafka topics, partitions, offsets, keys, and retention.
4. Study Spark DataFrames before Structured Streaming.
5. Study micro-batches, checkpoints, watermarks, and output modes.
6. Study Delta Lake Bronze, Silver, Gold, and time travel.
7. Study CPCB AQI breakpoints and interpolation.
8. Study PostgreSQL upserts and indexes.
9. Study Grafana panels and accessible labels.
10. Read testing, CI, and runbook documents before implementation.

[Back to README](../README.md)
