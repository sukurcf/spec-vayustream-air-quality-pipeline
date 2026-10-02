# System architecture

Purpose: This document explains the VayuStream architecture, main flows, local deployment, implementation repository shape, and required architecture decisions.

## Context diagram

```mermaid
flowchart LR
  Sensor["Default offline sensor simulator"] --> Kafka["Kafka KRaft broker"]
  OpenMeteo["Explicit opt-in Open-Meteo"] --> Kafka
  Engineer["Data engineer"] --> Runbook["Runbook and health checks"]
  Kafka --> Spark["PySpark Structured Streaming"]
  Spark --> Delta["Delta Lake Bronze, Silver, Quarantine"]
  Spark --> Postgres["PostgreSQL Gold and Alerts"]
  Poller["Python alert poller"] --> Postgres
  Poller --> AlertTopic["Kafka alerts topic"]
  AlertTopic --> Consumers["Alerting consumers"]
  Postgres --> CLI["Python inspection CLI: JSON, text, CSV"]
  Officer["Environment officer"] --> CLI
  Analyst["Data analyst"] --> CLI
  Runbook --> Spark
```

The system runs locally without internet after initial downloads. Simulated stations are the default Must source; Open-Meteo is separately opt-in Should scope. No student frontend is implemented. Built-in Kafka/Grafana consoles may be inspected separately for operations only, never as business or grader evidence.

## Container and component diagram

```mermaid
flowchart TB
  subgraph Local["Local Docker environment"]
    Producer["Python simulator and producer"]
    Kafka["Apache Kafka 4.3 KRaft broker"]
    Spark["One shared Spark driver and SparkSession: six queries"]
    DeltaStore["Local Delta storage"]
    Postgres["PostgreSQL 18"]
    CLI["Short-lived Python inspection CLI"]
    Poller["Python alert poller"]
    Health["Health check and task runner"]
  end
  Producer -->|"station_id key, JSON events"| Kafka
  Kafka -->|"raw topic"| Spark
  Spark -->|"append"| DeltaStore
  Spark -->|"provisional and final upserts"| Postgres
  Poller -->|"outbox and alert state"| Postgres
  Poller -->|"alert events"| Kafka
  CLI -->|"SELECT-only parameterized queries"| Postgres
  Health --> Kafka
  Health --> Spark
  Health --> Postgres
```

## Key sequence flows

### Simulator to Bronze

```mermaid
sequenceDiagram
  participant Sim as Simulator
  participant Kafka as Kafka raw topic
  participant Spark as Bronze query
  participant Delta as Bronze Delta
  Sim->>Kafka: publish reading with station_id key
  Kafka-->>Sim: acknowledgement or retry
  Spark->>Kafka: read micro-batch
  Spark->>Delta: append payload and Kafka metadata
  Spark-->>Spark: commit isolated checkpoint
```

### Silver validation and quarantine

```mermaid
sequenceDiagram
  participant Bronze as Bronze Delta
  participant Silver as Silver query
  participant Ref as Station reference
  participant DLQ as Kafka DLQ topic
  participant Quar as Quarantine Delta
  participant Valid as Silver Delta
  Silver->>Bronze: read raw batch
  Silver->>Silver: parse JSON and validate ranges
  Silver->>Ref: join station_id
  Silver->>Valid: write unique valid enriched events
  Silver->>Quar: store invalid event and reason
  Silver->>DLQ: publish invalid event
  Silver-->>Silver: commit checkpoint after all writes succeed
```

### Gold AQI and Python inspection

```mermaid
sequenceDiagram
  participant Silver as Silver valid readings
  participant Gold as Gold aggregation query
  participant PG as PostgreSQL
  participant CLI as Python inspection CLI
  Gold->>Silver: read event-time micro-batch
  Gold->>Gold: build 15-minute windows
  Gold->>Gold: calculate CPCB sub-index and max AQI
  Gold->>PG: upsert PROVISIONAL and FINAL window rows
  CLI->>PG: read AQI, windows, completeness, alerts
  PG-->>CLI: stored values and PROVISIONAL or FINAL status
  CLI-->>CLI: serialize exact JSON, text, or CSV
```

### Alert and recovery flow

```mermaid
sequenceDiagram
  participant City as City AQI windows
  participant Alert as Alert poller
  participant PG as PostgreSQL alerts
  participant Topic as Kafka alerts topic
  participant Consumer as Alerting consumer
  Alert->>City: poll FINAL city windows every 30 seconds
  City-->>Alert: two adjacent Poor or worse windows
  Alert->>PG: store state and pending outbox row
  Alert->>Topic: publish alert_id
  Alert->>PG: mark outbox row published
  Topic->>Consumer: at-least-once delivery
  City-->>Alert: FINAL AQI returns to 200 or less
  Alert->>PG: update recovery state and outbox
  Alert->>Topic: publish stable recovery alert
```

### Restart-safe replay

```mermaid
sequenceDiagram
  participant Kafka as Kafka
  participant Query as Spark query
  participant Sink as Delta or PostgreSQL sink
  participant Ckpt as Checkpoint
  Query->>Kafka: read offsets 40 to 50
  Query->>Sink: write idempotent batch
  Query--xCkpt: fail before checkpoint commit
  Query->>Kafka: restart from last committed offset
  Query->>Sink: replay same batch safely
  Query->>Ckpt: commit after successful sink write
```

## Data-flow description

1. The simulator creates JSON readings for 10 lite stations in 5 cities.
2. The producer sends each parseable reading to `vayu.readings.raw` with key `station_id`.
3. Bronze stores every Kafka message with topic, partition, offset, key, timestamp, payload, and `ingest_date`.
4. Silver parses JSON, casts fields, validates ranges, joins station reference data, deduplicates by `event_id`, and applies a 15-minute watermark.
5. Invalid events go to both `vayu.readings.dlq` and `quarantine.invalid_readings`.
6. Valid late events are dropped from Silver and counted. They do not go to the DLQ.
7. Gold creates 15-minute station and city windows. It calculates completeness and indicative AQI.
8. Gold upserts PROVISIONAL rows in Update mode and FINAL rows in Append mode.
9. The alert poller reads FINAL city windows, writes outbox rows, and publishes at-least-once alert messages.
10. The Python inspection CLI reads PostgreSQL serving tables only. It does not read raw Kafka/Delta or recompute AQI, and keeps null AQI and PROVISIONAL/FINAL status explicit.

## Local deployment view

| Profile | Laptop | Services | Proposed limits, not measurements |
|---|---|---|---|
| Lite | 4 cores, 8 GB RAM | One Kafka broker, one shared Spark driver, PostgreSQL, simulator/poller, short-lived CLI | Proposed aggregate 3392 MiB including Spark 2048 MiB/1280m heap and Python workers; no UI services. |
| Standard | 16 GB RAM recommended | Same core services, optional Mailpit | 50 sensors, 10 cities; proposed 6144 MiB including Mailpit; no required UI services. |

Windows users SHOULD use WSL2. For 8 GB laptops, set WSL memory to 4 GB and swap to 4 GB. For 16 GB laptops, set WSL memory to 8 GB and swap to 4 GB. The trainer MUST validate the lite profile on an 8 GB laptop before week 1.

[Doc 06](06-tech-stack-and-setup.md) is authoritative for fixed loopback ports, start/stop, database/topic/seed initialization, JAR caching, storage, and reset. Its RAM totals include all six queries in one driver plus Python/non-heap memory. Trainer pre-check must measure the implemented profile; no application has been run by this spec.

## Expected student repository tree

```text
vayustream/
  README.md
  pyproject.toml
  uv.lock
  .env.example
  src/vayustream/
    simulator/
    producer/
    streaming/
    transforms/
    aqi/
    alerts/
    inspection/
    ops/
  config/
    stations/
    spark/
  data/
    delta/
    checkpoints/
  tests/
    unit/
    integration/
    e2e/
    cli/
    local/
    fixtures/
  docs/
    adr/
    runbook.md
```

## Design principles

| Principle | VayuStream application |
|---|---|
| Medallion architecture | Bronze keeps raw audit data, Silver keeps trusted readings, and Gold serves SQL/CLI results and alerts. |
| Event-time processing | Watermarks and windows use `event_time`, not wall-clock arrival time. |
| Idempotent sinks | Delta and PostgreSQL use stable keys to survive replay. |
| At-least-once edges | Kafka and notifications can duplicate messages, so consumers deduplicate by stable IDs. |
| Pure transformations | Parsing, validation, AQI, and aggregation helpers support small batch DataFrame tests. |
| 12-factor configuration | Profiles, rates, paths, ports, and credentials come from environment variables. |
| Local-first operations | Lite targets an 8 GB/four-core laptop with no UI services, explicit simulator source, cached runtime dependencies, and persisted start/stop state. |
| Single-driver ownership | Six isolated-checkpoint queries share one SparkSession; alert poller and inspection are Python processes, not additional Spark drivers. |

## Required ADR topics

| ADR ID | Topic | Exact question the student MUST answer |
|---|---|---|
| ADR-001 | Spark and Delta version set | Which Spark, Delta Lake, Kafka connector, Scala, and Java versions are used, and why are they compatible? |
| ADR-002 | Checkpoint and replay strategy | How does each query choose checkpoint paths, and when is checkpoint deletion unsafe? |
| ADR-003 | PostgreSQL upsert design | Which business keys make station, city, and alert writes idempotent after replay? |
| ADR-004 | AQI interpretation | How does the project label 15-minute AQI as indicative while using CPCB breakpoints? |
| ADR-005 | Local hardware and CLI contract | How do the single driver, proposed all-process budgets, loopback ports, offline fixtures, SELECT-only inspector, and exact output contracts fit the lite target? |
| ADR-006 | Optional real-data mode | If Open-Meteo is enabled, how are terms, units, polling rate, and failures handled? |
| ADR-007 | Optional schema evolution | If Avro or object storage is added, which maintained tool is selected and why? |

[Back to README](../README.md)
