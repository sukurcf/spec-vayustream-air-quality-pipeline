# Pipeline specification

Purpose: This document defines the exact VayuStream pipeline interfaces, topics, jobs, schedules, contracts, errors, alerts, and runbook operations.

## Interface overview

| Interface item | Type | Main requirement IDs |
|---|---|---|
| Sensor producer | Kafka producer | FR-SIM-01, FR-SIM-02, FR-SIM-03, FR-KAF-02 |
| Kafka topics | Event transport | FR-KAF-01 |
| Bronze streaming query | Spark Structured Streaming | FR-BRZ-01, FR-OPS-01 |
| Silver streaming query | Spark Structured Streaming | FR-SLV-01, FR-SLV-02, FR-SLV-03, FR-SLV-04 |
| Gold streaming queries | Spark Structured Streaming and PostgreSQL | FR-GLD-01, FR-AQI-01, FR-GLD-02 |
| Alert poller | Restartable Python process and Kafka producer | FR-ALT-01, FR-ALT-02 |
| Grafana serving tables | PostgreSQL read interface | FR-DASH-01, FR-DASH-02 |
| Daily AQI batch | Should batch job | FR-DAILY-01 |
| Operations runbook | Human procedure | FR-OPS-03 |

## Kafka topic specification

| Topic name | Partitions | Key | Retention | Cleanup policy | Producer | Consumer | Notes |
|---|---:|---|---|---|---|---|---|
| `vayu.readings.raw` | 3 | `station_id` | 7 days minimum | `delete` | Simulator or Open-Meteo publisher | Bronze query | Required before simulator starts. |
| `vayu.readings.dlq` | 3 | `station_id` or `unknown` | 14 days minimum | `delete` | Silver query | Debug tools and optional replay | Contains invalid payloads only. |
| `vayu.alerts` | 3 | `alert_id` | 30 days minimum | `delete` | Alert poller | Alerting consumers and optional Mailpit worker | Delivery is at-least-once. |

Topic creation MUST be part of the documented one-command start. Health checks MUST fail with `topic-missing` when any required topic is absent.

## Producer requirements

The producer MUST use Apache Kafka idempotent producer settings. It MUST send each raw event with Kafka key equal to `station_id` when the value is parseable. If station ID is not parseable in a configured fault event, the key MAY be `unknown`.

Each event MUST include `schema_version`, `event_id`, `station_id`, `event_time`, `ingest_time`, `source`, `profile`, `pm25_ugm3`, `pm10_ugm3`, and `no2_ugm3`. The producer MUST preserve the same `event_id` for retries. It MUST flush or report pending records during graceful shutdown within 10 seconds.

The lite profile MUST publish readings for exactly 10 active stations in 5 Indian cities every 5 seconds. Seed `vayu-lite-001` MUST reproduce the deterministic 100-message fault scenario. The producer MUST never claim exactly-once delivery to Kafka consumers.

### Valid raw event example

```json
{
  "schema_version": "1.0",
  "event_id": "evt-20261002-del001-000001",
  "station_id": "DEL-001",
  "event_time": "2026-10-02T10:00:05Z",
  "ingest_time": "2026-10-02T10:00:06Z",
  "source": "simulator",
  "profile": "lite",
  "pm25_ugm3": 42.5,
  "pm10_ugm3": 88.2,
  "no2_ugm3": 34.0,
  "quality_flags": []
}
```

### Malformed fault example

```text
{"schema_version":"1.0","event_id":"evt-bad-001","station_id":"DEL-001","pm25_ugm3":
```

Expected result: Bronze stores the raw text. Silver writes one DLQ message and one quarantine row with reason `malformed-json`.

## Event field details

| Field | Type | Unit | Default | Required | Rejection reason |
|---|---|---|---|---|---|
| `schema_version` | string | none | none | Yes | `schema-version-unsupported` |
| `event_id` | string | none | generated stable ID | Yes | `event-id-invalid` |
| `station_id` | string pattern `AAA-999` | none | none | Yes | `required-field-missing` |
| `event_time` | UTC timestamp | ISO-8601 | none | Yes | `event-time-invalid` |
| `ingest_time` | UTC timestamp | ISO-8601 | send time | Yes | `ingest-time-invalid` |
| `source` | enum | none | `simulator` | Yes | `source-unsupported` |
| `profile` | enum | none | `lite` | Yes | `profile-unsupported` |
| `pm25_ugm3` | decimal | µg/m³ | none | Yes | `pollutant-out-of-range` |
| `pm10_ugm3` | decimal | µg/m³ | none | Yes | `pollutant-out-of-range` |
| `no2_ugm3` | decimal | µg/m³ | none | Yes | `pollutant-out-of-range` |
| `quality_flags` | string array | none | empty array | No | `quality-flag-unsupported` |

Allowed `source` values are `simulator` and `open-meteo`. Allowed `profile` values are `lite`, `standard`, and `real-data`. Must pollutant ranges are 0 to 1000 µg/m³.

## Streaming query specification

| Query name | Source | Sink | Trigger | Output mode | Watermark | Checkpoint path |
|---|---|---|---|---|---|---|
| `bronze_raw_ingest` | `vayu.readings.raw` | `bronze.raw_readings` Delta | Processing-time 10 seconds | Append | None | `data/checkpoints/bronze_raw_ingest` |
| `silver_validate_enrich` | `bronze.raw_readings` | `silver.valid_readings`, `quarantine.invalid_readings`, `vayu.readings.dlq` | Processing-time 10 seconds | Append | 15 minutes on `event_time` | Silver checkpoint path `data/checkpoints/silver_validate_enrich`. |
| `gold_station_windows_provisional` | `silver.valid_readings` | PostgreSQL `gold.station_window_metrics` | Processing-time 30 seconds | Update | 15 minutes on `event_time` | Provisional station checkpoint `data/checkpoints/gold_station_windows_provisional`. |
| `gold_city_aqi_provisional` | `silver.valid_readings` | PostgreSQL `gold.city_window_aqi` | Processing-time 30 seconds | Update | 15 minutes on `event_time` | Provisional city checkpoint `data/checkpoints/gold_city_aqi_provisional`. |
| `gold_station_windows_final` | `silver.valid_readings` | PostgreSQL `gold.station_window_metrics` | Processing-time 30 seconds | Append | 15 minutes on `event_time` | Final station checkpoint `data/checkpoints/gold_station_windows_final`. |
| `gold_city_aqi_final` | `silver.valid_readings` | PostgreSQL `gold.city_window_aqi` | Processing-time 30 seconds | Append | 15 minutes on `event_time` | `data/checkpoints/gold_city_aqi_final` |

The alert poller is not a Spark streaming query. It reads FINAL city rows from PostgreSQL every 30 seconds because JDBC is not a streaming source.

Each checkpoint path MUST be isolated. Sharing a checkpoint path MUST make health status `unhealthy` with reason `checkpoint-not-isolated`.

## Bronze interface

Bronze MUST append one row for each consumed raw Kafka message. It MUST store `bronze_id`, `topic`, `partition`, `offset`, `key`, `payload`, `kafka_timestamp`, `ingest_time`, and `ingest_date`. The audit key is `topic`, `partition`, and `offset`.

Example operation: if Kafka provides offset 51 on partition 0 with key `DEL-001`, Bronze writes one row with `topic=vayu.readings.raw`, `partition=0`, `offset=51`, and the original payload text. Duplicate event IDs remain as separate Bronze rows.

## Silver interface

Silver MUST parse JSON, cast types, validate required fields, validate pollutant ranges, join station reference data, drop duplicate `event_id` values within the watermark, and drop valid late events after the watermark.

Silver MUST use `dropDuplicatesWithinWatermark` on `event_id`. The watermark is the maximum observed valid event time minus 15 minutes. It never decreases and takes effect from the next micro-batch. Wall-clock time does not move the watermark. Silver drops events at or before the active cutoff.

Inside `foreachBatch`, Silver writes retry-safe targets in this order: Silver Delta, quarantine Delta, then DLQ topic. Silver and quarantine use Kafka topic, partition, and offset keys or Delta idempotent batch options. If a crash happens between two sink writes, replay updates existing rows and republishes possible duplicate DLQ messages with stable source metadata.

| Outcome | Sink | Counted metric | Example reason |
|---|---|---|---|
| Valid unique event | `silver.valid_readings` | `silver_rows_written` | none |
| Malformed JSON | DLQ and quarantine | `invalid_events_total` | `malformed-json` |
| Missing field | DLQ and quarantine | `invalid_events_total` | `required-field-missing` |
| Out-of-range pollutant | DLQ and quarantine | `invalid_events_total` | `pollutant-out-of-range` |
| Unknown station | DLQ and quarantine | `invalid_events_total` | `station-not-found` |
| Duplicate event | no sink | `duplicate_events_dropped` from Spark state-operator progress | duplicate `event_id` |
| Valid late event | no sink | `late_events_dropped` | watermark exceeded |

Example operation: a valid JSON event with `no2_ugm3=-1` creates no Silver row. It creates one DLQ message and one quarantine row with reason `pollutant-out-of-range`.

## Gold interface and upsert keys

Gold station windows MUST use key `station_id` and `window_start`. Gold city windows MUST use key `city` and `window_start`. Alert rows MUST use key `alert_id`. Replayed batches MUST update the existing row instead of creating a duplicate.

Gold writes the same PostgreSQL tables from 2 sink modes. Update-mode queries write `PROVISIONAL` rows for fast Grafana panels. Append-mode queries update the same rows to `FINAL` once `window_end` is less than or equal to the watermark. A later PROVISIONAL write MUST NOT overwrite a FINAL row.

| Sink | Business key | Required operation | Duplicate result |
|---|---|---|---|
| `gold.station_window_metrics` | `station_id`, `window_start` | Idempotent upsert with FINAL protection | Row is updated. |
| `gold.city_window_aqi` | `city`, `window_start` | Idempotent upsert with FINAL protection | Row is updated. |
| `alerts.alert_events` | `alert_id` | Idempotent upsert | Row is updated. |

Example operation: city `Delhi` and `window_start=2026-10-02T10:00:00Z` replay twice. PostgreSQL contains exactly one `gold.city_window_aqi` row for that key after both attempts.

## AQI calculation interface

The AQI function input is one city window with pollutant averages. It MUST calculate one sub-index for each available pollutant. It MUST return the maximum sub-index as `aqi_value`. It MUST return null when fewer than 3 pollutants are available or when neither PM2.5 nor PM10 is available.

Formula in plain text: sub_index = ((index_high - index_low) / (breakpoint_high - breakpoint_low)) * (concentration - breakpoint_low) + index_low.

Before band lookup, truncate CO to 1 decimal mg/m³ and all other pollutants to whole µg/m³. Round every sub-index half up. Concentration 0 returns sub-index 0.

| Category | AQI range | PM2.5 | PM10 | NO2 | SO2 | CO | O3 | NH3 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Good | 0-50 | 0-30 | 0-50 | 0-40 | 0-40 | 0-1.0 | 0-50 | 0-200 |
| Satisfactory | 51-100 | 31-60 | 51-100 | 41-80 | 41-80 | 1.1-2.0 | 51-100 | 201-400 |
| Moderate | 101-200 | 61-90 | 101-250 | 81-180 | 81-380 | 2.1-10 | 101-168 | 401-800 |
| Poor | 201-300 | 91-120 | 251-350 | 181-280 | 381-800 | 10.1-17 | 169-208 | 801-1200 |
| Very Poor | 301-400 | 121-250 | 351-430 | 281-400 | 801-1600 | 17.1-34 | 209-748 | 1201-1800 |
| Severe | 401-500 | >250 | >430 | >400 | >1600 | >34 | >748 | >1800 |

Units are µg/m³ except CO, which uses mg/m³. The official Severe band is open-ended. VayuStream continues the Very Poor slope and caps at 500. Effective Severe caps are PM2.5 380.3, PM10 509.8, NO2 520.2, SO2 2407.1, CO 51.1, O3 1292.4, and NH3 2405.1. This cap is a VayuStream convention, not an official CPCB value.

Example operation: PM2.5 is 95 µg/m³, PM10 is 260 µg/m³, and NO2 is 90 µg/m³. The sub-indexes are PM2.5 215, PM10 210, and NO2 110. The function returns AQI 215, category `Poor`, and basis `15-minute indicative`.

## Alert flow

An unhealthy alert starts when a city is Poor or worse for 2 adjacent FINAL 15-minute windows. Poor starts at AQI 201. Cooldown is 60 minutes from `created_at` for each city and category. Recovery starts when a city with an active unhealthy alert returns to AQI 200 or less. A gap breaks the previous streak. An unhealthy window after the gap starts a new streak.

The poller persists its cursor, streak, cooldown, and outbox state in PostgreSQL. It first records a pending outbox row, then publishes to `vayu.alerts`, then marks the row published. A crash cannot lose the alert. Duplicate Kafka messages remain possible.

| Step | Input condition | Required output |
|---|---|---|
| Watch | First Poor, Very Poor, or Severe city window | Store watch state only. |
| Trigger | Second adjacent FINAL city window has AQI greater than 200 | Upsert unhealthy alert, create outbox row, and publish to `vayu.alerts`. |
| Cooldown | Same city and category stays unhealthy inside 60 minutes | Do not publish another unhealthy alert. |
| New category | Same city reaches 2 adjacent windows in a different unhealthy category | Publish a new category alert even during the previous category cooldown. |
| Recovery | Active alert city has AQI 200 or less | Upsert recovery alert and publish recovery event. |

Alert event example:

```json
{
  "schema_version": "1.0",
  "alert_id": "alt-delhi-poor-20261002T101500Z",
  "city": "Delhi",
  "alert_type": "UNHEALTHY",
  "aqi_category": "Poor",
  "aqi_value": 245,
  "trigger_window_start": "2026-10-02T10:15:00Z",
  "trigger_window_end": "2026-10-02T10:30:00Z",
  "cooldown_until": "2026-10-02T11:30:15Z",
  "created_at": "2026-10-02T10:30:15Z"
}
```

Expected result: consumers can receive the event more than once. They MUST store one incident by `alert_id`.

## Error format

Pipeline commands and health checks MUST report errors with stable identifiers. Log records SHOULD be JSON. The error format has these fields.

| Field | Type | Required | Example |
|---|---|---|---|
| `error_id` | string | Yes | `topic-missing` |
| `message` | string | Yes | Required topic `vayu.readings.raw` does not exist. |
| `component` | string | Yes | `health-check` |
| `correlation_id` | string | Yes | `run-20261002-001` |
| `details` | object | No | `{"topic":"vayu.readings.raw"}` |

Example:

```json
{
  "error_id": "checkpoint-not-isolated",
  "message": "Query gold_city_aqi shares a checkpoint path.",
  "component": "health-check",
  "correlation_id": "run-20261002-001",
  "details": {
    "checkpoint_path": "data/checkpoints/shared"
  }
}
```

## Health check interface

| Check | Success condition | Failure identifier |
|---|---|---|
| Kafka broker | Broker accepts metadata request. | `kafka-unavailable` |
| Kafka topics | All 3 required topics exist. | `topic-missing` |
| Spark queries | Required query names are running or cleanly stopped. | `query-not-running` |
| Alert poller | Poller heartbeat is newer than 60 seconds. | `alert-poller-stale` |
| Checkpoints | Query checkpoint paths are unique. | `checkpoint-not-isolated` |
| PostgreSQL | Connection succeeds and serving tables exist. | `postgres-unavailable` |
| Grafana | Data source points to PostgreSQL. | `grafana-datasource-invalid` |
| Lite profile | Kafka UI is disabled and Spark limit is 2304 MiB with 1500m heap. | `lite-service-disabled` |

Example operation: Kafka is down at startup. Health status is `unhealthy`, failure reason is `kafka-unavailable`, and the simulator MUST not publish events.

## Deterministic test interface

The deterministic end-to-end test MUST use named, ordered micro-batches, fixed timestamps, isolated checkpoints, and an explicit output mode. It MUST process exactly this schedule with seed `vayu-lite-001`.

1. B1 has 97 records. It has 86 unique valid readings in the 10:00-10:15 UTC window across all 10 stations, 5 extra duplicate copies, and 6 invalid messages.
2. B2 has 1 new valid reading for `DEL-001` at 10:31 UTC. It moves the Silver watermark to 10:16.
3. After the watermark is active, B3 has 2 new valid event IDs with event time 10:00 UTC. Silver drops them as late.

| Input category | Count | Expected handling |
|---|---:|---|
| Raw Kafka messages | 100 | Bronze rows = 100. |
| Malformed JSON | 3 | DLQ rows = 3 and quarantine rows = 3. |
| Missing required fields | 2 | DLQ rows = 2 and quarantine rows = 2. |
| Out-of-range pollutant | 1 | DLQ rows = 1 and quarantine rows = 1. |
| Extra duplicate copies | 5 | Dropped after validation. |
| Valid late events | 2 | Dropped after watermark, not DLQ. |
| Final Silver rows | 87 | 100 - 6 invalid - 5 extra duplicate copies - 2 late. |
| FINAL Gold station windows | 10 | One FINAL 10:00 row per lite station. |
| FINAL Gold city windows | 5 | One FINAL 10:00 row per lite city. |
| Total Gold station windows | 11 | FINAL 10:00 rows plus one PROVISIONAL 10:30 station row. |
| Total Gold city windows | 6 | FINAL 10:00 rows plus one PROVISIONAL 10:30 city row. |

## Should daily AQI batch

The daily batch SHOULD write `gold.daily_city_aqi`. It SHOULD run once per day after the UTC day closes. It SHOULD use 24-hour averages for PM2.5, PM10, NO2, SO2, and NH3. It SHOULD use maximum 8-hour averages for CO. This is the VayuStream reading of the CPCB O3 footnote: when maximum 8-hour average O3 is 208 µg/m³ or less, use it with Good through Poor bands; when it is more than 208 µg/m³, use maximum 1-hour O3 with Very Poor and Severe bands. It SHOULD require at least 16 valid hourly values for 24-hour pollutants and at least 6 valid hourly values for each 8-hour CO or O3 block.

O3 example A: maximum 8-hour O3 is 190 µg/m³ and maximum 1-hour O3 is 260 µg/m³. The batch uses the 8-hour basis and returns sub-index 254. O3 example B: maximum 8-hour O3 is 230 µg/m³ and maximum 1-hour O3 is 400 µg/m³. The batch uses the 1-hour basis and returns sub-index 336.

| Field | Type | Unit | Required |
|---|---|---|---|
| `city` | string | none | Yes |
| `aqi_date` | date | UTC date | Yes |
| `aqi_value` | integer nullable | AQI | Yes |
| `aqi_category` | enum nullable | none | Yes |
| `dominant_pollutant` | string nullable | none | Yes |
| `data_completeness_pct` | decimal | percent | Yes |
| `basis` | string | none | Yes, value `daily official-style` |

Example operation: Delhi has complete 24-hour PM2.5, PM10, and NO2 data for 2026-10-02. The batch writes one Delhi row with `basis=daily official-style`.

## Runbook procedures

### Restart a streaming query

1. Stop only the failed query when possible.
2. Keep the query checkpoint directory.
3. Verify Kafka, PostgreSQL, and Delta paths are healthy.
4. Start the same query name with the same checkpoint path.
5. Confirm the next progress log has no skipped committed offset.

Expected result: a Silver query stopped after offset 50 restarts and processes offset 51 or the replayed uncommitted batch.

### Replay from Kafka offsets

1. Record the topic, partition, start offset, and end offset.
2. Confirm downstream sinks have idempotent keys.
3. Use an isolated replay checkpoint path.
4. Replay into a clearly named replay run.
5. Compare row counts and keys before merging results.

Expected result: replaying Delhi offsets 10 to 30 does not create duplicate gold rows because city and station keys include `window_start`.

### Reprocess Bronze to Silver

1. Freeze the affected Bronze date range.
2. Keep the original Bronze table unchanged.
3. Use a new Silver checkpoint path for the reprocess run.
4. Write to a reviewed Silver target or replace only after counts match.
5. Rebuild dependent Gold windows for the same event-time range.

Expected result: reprocessing `ingest_date=2026-10-02` corrects Silver validation logic without deleting Bronze audit rows.

## Failure handling and SLA

| Failure | Handling | Student evidence |
|---|---|---|
| Producer retry | Reuse same `event_id`; preserve station key. | Producer test and logs. |
| Invalid payload | Send to DLQ and quarantine. | DLQ and quarantine count checks. |
| PostgreSQL outage | Retry batch and keep checkpoint uncommitted. | Restart-recovery test. |
| Query crash | Restart from isolated checkpoint. | Runbook demo. |
| Duplicate alert delivery | Consumer deduplicates by `alert_id`. | Alert consumer test. |

Lite p95 ingest-to-dashboard latency MUST be 120 seconds or less. Producer throughput MUST sustain 2 readings per second for 10 minutes.

[Back to README](../README.md)
