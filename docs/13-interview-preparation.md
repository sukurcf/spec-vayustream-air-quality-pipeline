# Interview preparation

Purpose: This document helps the student explain VayuStream clearly in interviews and viva sessions.

## Explain your project in 2 minutes

Use STAR.

| Part | Talking points |
|---|---|
| Situation | Indian city air-quality teams need faster signals than daily reports. Raw sensor data has duplicates, bad values, late events, and gaps. |
| Task | I built VayuStream, a local streaming pipeline for indicative 15-minute AQI and repeat-safe alerts. |
| Action | I used Kafka 4, PySpark Structured Streaming, Delta Lake, PostgreSQL, and Grafana. I designed Bronze, Silver, Gold, quarantine, DLQ, and alerts. |
| Result | In lite mode, 10 sensors run on an 8 GB laptop. Tests prove deterministic faults, AQI boundaries, restart safety, and coverage gates. |

## Interview questions by topic

### Kafka

| # | Question | Key points to cover |
|---:|---|---|
| 1 | What is a Kafka topic in VayuStream? | Named stream; `vayu.readings.raw`, `vayu.readings.dlq`, `vayu.alerts`; retention; partitions. |
| 2 | Why did you key raw messages by station ID? | Station ordering; same station to same partition; easier debugging; stable join key. |
| 3 | What is an offset? | Position in a partition; consumer progress; replay control; checkpoint relationship. |
| 4 | What is a consumer group? | Shared consumption; partitions assigned to consumers; Spark query uses group semantics. |
| 5 | How does retention affect replay? | Raw topic keeps at least 7 days; replay possible before deletion; storage trade-off. |
| 6 | What does idempotent producer mean? | Retries avoid duplicate records in some producer failure cases; stable `event_id`; not exactly-once for whole pipeline. |

### Spark and Structured Streaming

| # | Question | Key points to cover |
|---:|---|---|
| 7 | What is lazy evaluation in Spark? | Transformations build a plan; actions start execution; optimizer can rearrange work. |
| 8 | What is a micro-batch? | Small batch per trigger; processes new Kafka offsets; records progress and checkpoint state. |
| 9 | Why use watermarks? | Bound late data state; 15-minute watermark; drop very late valid events; count drops. |
| 10 | What is a stream-static join? | Streaming readings join station CSV; reference is static; unknown stations are rejected. |
| 11 | What causes a shuffle? | GroupBy window and city; joins may shuffle; partitions affect performance. |
| 12 | What are output modes? | Append, update, complete; window query choice; affects sink writes and duplicates. |
| 13 | Why design transformations as pure functions? | Batch unit tests; deterministic inputs; less I/O coupling; easier debugging. |

### Delta Lake and data quality

| # | Question | Key points to cover |
|---:|---|---|
| 14 | Why use Bronze, Silver, and Gold? | Audit raw data; trusted typed data; analytics-ready windows; clear responsibilities. |
| 15 | What does ACID mean for Delta? | Atomic writes; consistent metadata; isolated readers; durable table history. |
| 16 | What is time travel useful for? | Debug bad batch; compare table versions; support reprocessing decisions. |
| 17 | What goes to quarantine? | Malformed JSON; missing required fields; out-of-range pollutants; station not found. |
| 18 | What goes to DLQ? | Invalid events for debugging; original payload; reason code; station key or unknown. |

### AQI and alerting

| # | Question | Key points to cover |
|---:|---|---|
| 19 | How is the sub-index calculated? | CPCB breakpoints; linear interpolation; pollutant concentration range; maximum sub-index. |
| 20 | When is AQI null? | Fewer than 3 pollutants; no PM2.5 or PM10; insufficient data is not zero. |
| 21 | Why is official AQI different? | Uses 24-hour averages; CO and O3 use 8-hour average; VayuStream is 15-minute indicative. |
| 22 | How is an unhealthy alert triggered? | Poor or worse; 2 consecutive 15-minute windows; stable alert ID. |
| 23 | How does cooldown work? | 60 minutes; per city and category; prevents repeated alerts. |

### SQL, PostgreSQL, and Grafana

| # | Question | Key points to cover |
|---:|---|---|
| 24 | Why use PostgreSQL for Gold? | Grafana data source; SQL queries; unique keys; local free tool. |
| 25 | What is an idempotent upsert? | Same business key updates one row; safe replay; no duplicate city-window rows. |
| 26 | Which indexes matter? | City plus window start; station plus window start; alert ID; dashboard time filters. |
| 27 | How does Grafana show null AQI? | Text says `Insufficient data`; labels plus colours; accessibility. |

### Testing and DevOps

| # | Question | Key points to cover |
|---:|---|---|
| 28 | What makes the 100-message test deterministic? | Seed `vayu-lite-001`; fixed timestamps; fixed batch schedule; isolated checkpoint. |
| 29 | How do you test restart recovery? | Fail after sink write; restart with checkpoint; assert one PostgreSQL row per key. |
| 30 | What quality gates block merge? | Ruff; mypy; tests; coverage; Gitleaks; pip-audit; Trivy; image build. |
| 31 | Why pin Docker images? | Reproducible grading; avoid surprise version changes; match compatibility matrix. |
| 32 | What would you monitor in production? | Consumer lag; micro-batch duration; input and output counts; DLQ rate; alert count. |

## Project deep-dive questions

| # | Question | Key points to cover |
|---:|---|---|
| 1 | How did you choose the lite profile limits? | 8 GB RAM; 1 broker; Spark 2304 MiB; 10 sensors; no Kafka UI. |
| 2 | How do malformed JSON events move through the pipeline? | Bronze keeps raw payload; Silver parse fails; quarantine row; DLQ message. |
| 3 | How do duplicates affect Bronze, Silver, and Gold? | Bronze keeps both; Silver dedupes by `event_id`; Gold reading count excludes duplicate. |
| 4 | How do you calculate completeness for Delhi? | Unique valid readings; expected readings; 15 minutes; active sensors; rounded percent. |
| 5 | Why are alerts stored in PostgreSQL and Kafka? | PostgreSQL for dashboard; Kafka for consumers; same `alert_id`; at-least-once delivery. |
| 6 | What happens if PostgreSQL is down? | Sink fails or retries; health degraded; checkpoint prevents silent loss; runbook recovery. |
| 7 | How do you reprocess Bronze to Silver? | Choose date range; preserve raw data; write to isolated checkpoint; compare counts. |
| 8 | What is the hardest Spark bug you debugged? | Explain symptom; plan; metric or log used; fix; test added. |
| 9 | How would standard mode change performance? | 50 sensors; 10 readings/sec; more partitions; Kafka UI; 16 GB RAM. |

## Fundamentals check

### Python core

- Explain lists, dictionaries, dataclasses, and type hints with VayuStream examples.
- Explain exceptions used for validation and graceful shutdown.
- Explain context managers for files and clients.
- Explain why pure functions make AQI and validation tests easier.

### SQL

- Explain inner join versus left join using station reference and readings.
- Explain `GROUP BY` for city windows and station windows.
- Explain window functions for trend ranking or latest AQI per city.
- Explain transactions and why upserts need unique keys.
- Explain indexes on `(city, window_start)` and `(station_id, window_start)`.

### Git

- Explain branch, commit, pull request, merge, rebase, and tag.
- Explain Conventional Commits with a VayuStream example.
- Explain how to fix a failed CI check before merge.

### Linux

- Explain `cd`, `ls`, `grep`, `tail`, `chmod`, `ps`, and environment variables.
- Explain how to read container logs for a failed Spark query.
- Explain why WSL2 files should live inside the Linux filesystem.

### HTTP and networking

- Explain TCP, DNS, ports, localhost, and Docker bridge networking.
- Explain HTTP status codes used by health endpoints.
- Explain why Open-Meteo polling needs timeout and retry rules.

### Local tools mapped to AWS services

| Local VayuStream tool | Similar AWS service | Interview note |
|---|---|---|
| Kafka | Amazon MSK | Managed Kafka still uses topics, partitions, offsets, and consumer groups. |
| Delta files | Amazon S3 plus table format | Cloud storage changes operations, not medallion concepts. |
| Spark local | AWS Glue or EMR | Same Spark DataFrame and Structured Streaming concepts apply. |
| PostgreSQL | Amazon RDS for PostgreSQL | Upserts, indexes, and SQL skills transfer. |
| Grafana OSS | Amazon Managed Grafana | Dashboard design and queries transfer. |
| Docker Compose | ECS task or Kubernetes pod idea | Containers package services; orchestration differs. |

## Resume bullets

- Built `vayustream`, a Kafka 4 and PySpark Structured Streaming pipeline that processed `[N]` synthetic air-quality events with `[P95]` seconds p95 latency.
- Designed Bronze, Silver, Quarantine, and Gold layers with Delta Lake and PostgreSQL, preventing duplicate business keys during replay.
- Implemented CPCB indicative AQI calculation with 100% branch coverage across breakpoint boundary tests.
- Created deterministic streaming tests with 100 messages, 5 extra duplicate copies, 6 invalid records, and 2 late drops.
- Delivered Grafana dashboards for current AQI, AQI trend, and alerts, with labels for accessible interpretation.

## GitHub and LinkedIn tips

- Pin the `vayustream` repository on GitHub.
- Keep the README short, reproducible, and screenshot-free unless images are truly useful.
- Link the final release and demo video.
- Write a LinkedIn post that explains one hard lesson about watermarks or idempotent upserts.
- Do not claim production AQI accuracy. Say `indicative 15-minute AQI`.

## Mock-interview checklist

- Explain the project in 2 minutes without reading notes.
- Draw the Kafka to Bronze to Silver to Gold flow on paper.
- Explain one AQI boundary test and one restart test.
- Show the latest CI run and coverage report.
- Explain one trade-off from an ADR.
- Run one SQL query against a gold table.
- Answer why at-least-once delivery can create duplicates.
- Explain what you would improve with one more week.

[Back to README](../README.md)
