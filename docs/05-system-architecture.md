# System architecture

Purpose: This document explains the VayuStream architecture, main flows, local deployment, implementation repository shape, and required architecture decisions.

## Context diagram

```mermaid
flowchart LR
  Sensor["Sensor simulator"] --> Kafka["Kafka KRaft broker"]
  OpenMeteo["Open-Meteo mode"] --> Kafka
  Engineer["Data engineer"] --> Runbook["Runbook and health checks"]
  Kafka --> Spark["PySpark Structured Streaming"]
  Spark --> Delta["Delta Lake Bronze, Silver, Quarantine"]
  Spark --> Postgres["PostgreSQL Gold and Alerts"]
  Poller["Python alert poller"] --> Postgres
  Poller --> AlertTopic["Kafka alerts topic"]
  AlertTopic --> Consumers["Alerting consumers"]
  Postgres --> Grafana["Grafana dashboard"]
  Officer["Environment officer"] --> Grafana
  Analyst["Data analyst"] --> Postgres
  Runbook --> Spark
```

The system runs locally. It uses simulated air-quality stations as the Must source. Open-Meteo is a Should source only.

## Container and component diagram

```mermaid
flowchart TB
  subgraph Local["Local Docker environment"]
    Producer["Python simulator and producer"]
    Kafka["Apache Kafka 4.3 KRaft broker"]
    Spark["Spark driver with PySpark jobs"]
    DeltaStore["Local Delta storage"]
    Postgres["PostgreSQL 18"]
    Grafana["Grafana OSS 13"]
    Poller["Python alert poller"]
    Health["Health check and task runner"]
  end
  Producer -->|"station_id key, JSON events"| Kafka
  Kafka -->|"raw topic"| Spark
  Spark -->|"append"| DeltaStore
  Spark -->|"provisional and final upserts"| Postgres
  Poller -->|"outbox and alert state"| Postgres
  Poller -->|"alert events"| Kafka
  Grafana -->|"SQL queries"| Postgres
  Health --> Kafka
  Health --> Spark
  Health --> Postgres
  Health --> Grafana
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

### Gold AQI and dashboard

```mermaid
sequenceDiagram
  participant Silver as Silver valid readings
  participant Gold as Gold aggregation query
  participant PG as PostgreSQL
  participant Grafana as Grafana
  Gold->>Silver: read event-time micro-batch
  Gold->>Gold: build 15-minute windows
  Gold->>Gold: calculate CPCB sub-index and max AQI
  Gold->>PG: upsert PROVISIONAL and FINAL window rows
  Grafana->>PG: read current AQI, trend, alerts
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
10. Grafana reads PostgreSQL only. It does not read directly from Kafka or Delta.

## Local deployment view

| Profile | Laptop | Services | Required limits |
|---|---|---|---|
| Lite | 8 GB RAM | Kafka, Spark local mode, PostgreSQL, Grafana, simulator | Kafka 768 MiB, Spark 2304 MiB, PostgreSQL 384 MiB, Grafana 192 MiB, simulator 128 MiB, Kafka UI disabled. |
| Standard | 16 GB RAM | Lite services plus Kafka UI and optional Mailpit | 50 sensors, 10 cities, Spark driver memory chosen by ADR, Kafka UI enabled. |

Windows users SHOULD use WSL2. For 8 GB laptops, set WSL memory to 4 GB and swap to 4 GB. For 16 GB laptops, set WSL memory to 8 GB and swap to 4 GB. The trainer MUST validate the lite profile on an 8 GB laptop before week 1.

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
    ops/
  config/
    stations/
    grafana/
    spark/
  data/
    delta/
    checkpoints/
  tests/
    unit/
    integration/
    e2e/
    fixtures/
  docs/
    adr/
    runbook.md
```

## Design principles

| Principle | VayuStream application |
|---|---|
| Medallion architecture | Bronze keeps raw audit data, Silver keeps trusted readings, and Gold serves dashboard tables. |
| Event-time processing | Watermarks and windows use `event_time`, not wall-clock arrival time. |
| Idempotent sinks | Delta and PostgreSQL use stable keys to survive replay. |
| At-least-once edges | Kafka and notifications can duplicate messages, so consumers deduplicate by stable IDs. |
| Pure transformations | Parsing, validation, AQI, and aggregation helpers support small batch DataFrame tests. |
| 12-factor configuration | Profiles, rates, paths, ports, and credentials come from environment variables. |
| Local-first operations | The lite stack runs on an 8 GB laptop without cloud services. |

## Required ADR topics

| ADR ID | Topic | Exact question the student MUST answer |
|---|---|---|
| ADR-001 | Spark and Delta version set | Which Spark, Delta Lake, Kafka connector, Scala, and Java versions are used, and why are they compatible? |
| ADR-002 | Checkpoint and replay strategy | How does each query choose checkpoint paths, and when is checkpoint deletion unsafe? |
| ADR-003 | PostgreSQL upsert design | Which business keys make station, city, and alert writes idempotent after replay? |
| ADR-004 | AQI interpretation | How does the project label 15-minute AQI as indicative while using CPCB breakpoints? |
| ADR-005 | Local hardware profile | Which services are disabled or limited in lite mode to fit 8 GB RAM? |
| ADR-006 | Optional real-data mode | If Open-Meteo is enabled, how are terms, units, polling rate, and failures handled? |
| ADR-007 | Optional schema evolution | If Avro or object storage is added, which maintained tool is selected and why? |

[Back to README](../README.md)
