# Project overview

Purpose: This document explains why VayuStream exists, what the student must build, and how success is measured.

## Problem statement

Indian city air-quality teams need faster signals than daily reports. Low-cost stations can send frequent readings, but raw readings include duplicates, bad values, late data, and gaps. VayuStream defines a local streaming pipeline that ingests these readings, cleans them, computes 15-minute indicative AQI windows, and raises repeat-safe city alerts.

The project is a portfolio specification for one fresher. It targets data engineering roles that ask for Kafka, PySpark Structured Streaming, Delta Lake, PostgreSQL, and Grafana.

## Vision

VayuStream MUST let a student demonstrate an end-to-end streaming data product on a laptop. The system MUST be small enough for 8 GB RAM in lite mode. It SHOULD also show stronger standard-mode features on 16 GB RAM.

## Measurable goals

| Goal ID | Goal | Measure |
|---|---|---|
| G-01 | Ingest simulated station events | 10 lite sensors publish one event every 5 seconds to `vayu.readings.raw`. |
| G-02 | Preserve audit history | Bronze Delta keeps every raw Kafka message with metadata and `ingest_date`. |
| G-03 | Produce trusted readings | Silver keeps valid unique events and rejects invalid events to DLQ and quarantine. |
| G-04 | Compute indicative AQI | City windows use CPCB breakpoints and return null when data is insufficient. |
| G-05 | Alert safely | Poor or worse AQI for 2 consecutive windows creates one stable alert per cooldown. |
| G-06 | Show operations value | Grafana shows current AQI, AQI trend, and alert history from PostgreSQL. |
| G-07 | Prove quality | Tests meet line coverage 80% and branch coverage 70%. |

## Scope by priority

### MVP Must scope

- Kafka 4.x KRaft with topics `vayu.readings.raw`, `vayu.readings.dlq`, and `vayu.alerts`.
- Sensor simulator for 10 sensors in 5 Indian cities.
- JSON event contract with event ID, event time, ingest time, units, and source.
- Bronze, Silver, Quarantine, Gold, and Alerts data layers.
- 15-minute tumbling windows and a 15-minute watermark.
- CPCB National AQI sub-index method for PM2.5, PM10, and NO2.
- PostgreSQL idempotent upserts for gold and alerts.
- Grafana dashboard with at least 3 panels.
- One-command local start, health checks, and runbook procedures.

### Should scope

- 50 sensors in 10 cities and the full pollutant set.
- 1-hour windows and Delta gold copies.
- Open-Meteo air-quality polling once per hour for each city.
- Mailpit or webhook notifications from `vayu.alerts`.
- Daily official-style AQI batch table `gold.daily_city_aqi`.
- Kafka UI, streaming progress metrics, and Delta maintenance.

### Could scope

- Avro with an open-source schema registry.
- Maintained S3-compatible object storage for Delta experiments.
- Sensor-drift anomaly detection with z-scores.
- Grafana Geomap panel.

## Non-goals and out of scope

- The student MUST NOT connect to real sensor hardware.
- The project MUST NOT include a mobile app.
- The project MUST NOT build machine-learning forecasts.
- The project MUST NOT require a cloud account or managed Kafka.
- The project MUST NOT promise exactly-once delivery for e-mail, webhooks, or Kafka notifications.
- The project MUST NOT collect personal data.

## Assumptions

| ID | Assumption |
|---|---|
| A-01 | The trainer provides this specification repository. |
| A-02 | The student creates a public implementation repository named `vayustream`. |
| A-03 | Docker runs locally on Windows 11 with WSL2, macOS, or Linux. |
| A-04 | Lite mode uses synthetic data only. |
| A-05 | Open-Meteo mode is optional and non-commercial. |
| A-06 | The official CPCB AQI method is used as the reference method. |

## Constraints

| ID | Constraint |
|---|---|
| C-01 | MUST work with 8 GB RAM in lite profile. |
| C-02 | SHOULD be developed on 16 GB RAM for standard profile. |
| C-03 | MUST use free local tools for all mandatory work. |
| C-04 | MUST use Python 3.12.x and uv. |
| C-05 | MUST use Apache Kafka 4.x KRaft and PySpark 4.2.x. |
| C-06 | MUST pin Docker image tags and dependency minor lines. |
| C-07 | MUST keep official AQI claims honest. The real-time AQI is indicative. |

## Effort budget

The full project is planned for 6 weeks and about 180 hours. The MUST scope MUST fit in about 120 hours. This includes about 30 hours of guided learning and setup in week 1. SHOULD items, hardening, interview preparation, and contingency use the remaining about 60 hours. COULD items start only after all MUST items pass.

## Success criteria

| ID | Criterion | Evidence |
|---|---|---|
| SC-01 | Lite stack starts from a clean clone. | README setup demo in 10 steps or fewer. |
| SC-02 | Sensor simulator creates deterministic faults. | Test data includes 100 named events with fixed timestamps. |
| SC-03 | Bronze has no missed raw Kafka messages during normal run. | Row counts match consumed raw messages. |
| SC-04 | Silver rejects malformed and out-of-range events. | DLQ topic and quarantine table show exact reasons. |
| SC-05 | PostgreSQL gold tables have no duplicate business keys after replay. | Restart-recovery test passes. |
| SC-06 | Alert cooldown prevents alert spam. | Demo shows one alert in 60 minutes for the same city and category. |
| SC-07 | Coverage gates pass. | CI report shows line coverage at least 80% and branch coverage at least 70%. |

## Skills learned and job relevance

| Skill | Job-description keywords |
|---|---|
| Kafka topic design | Kafka, KRaft, partitions, offsets, retention, producer idempotence |
| PySpark Structured Streaming | micro-batch, watermark, window, stream-static join, checkpoint |
| Delta Lake medallion design | bronze, silver, gold, ACID, time travel |
| Data quality | schema validation, quarantine, DLQ, deduplication |
| AQI calculation | domain rules, linear interpolation, boundary testing |
| PostgreSQL analytics sink | upsert, unique key, dashboard query, idempotency |
| Grafana dashboarding | datasource provisioning, panels, operational visibility |
| Testing streaming systems | deterministic micro-batches, restart recovery, coverage gates |

## Related documents

- [Users and roles](02-users-and-roles.md)
- [Functional requirements](03-functional-requirements.md)
- [Non-functional requirements](04-non-functional-requirements.md)
- [System architecture](05-system-architecture.md)
- [Tech stack and setup](06-tech-stack-and-setup.md)
- [Data model](07-data-model.md)
- [Pipeline specification](08-pipeline-specification.md)
- [Testing strategy](09-testing-strategy-and-test-cases.md)

[Back to README](../README.md)
