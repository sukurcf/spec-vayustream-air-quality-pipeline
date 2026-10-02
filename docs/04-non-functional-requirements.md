# Non-functional requirements

Purpose: This document defines measurable quality targets for VayuStream and how the student verifies them.

## NFR catalog

| ID | Category | Priority | Target | Verification method |
|---|---|---|---|---|
| NFR-PERF-01 | Performance | Must | Lite p95 ingest-to-inspectable PROVISIONAL PostgreSQL gold latency is 120 seconds or less. FINAL readiness depends on the event-time watermark, not this latency target. | Run 10 sensors for 15 minutes and compare Kafka timestamp with PostgreSQL visible time through CLI/SQL. |
| NFR-PERF-02 | Performance | Must | Producer sustains 2 readings per second for 10 minutes with zero unhandled send errors. | Load test in lite profile. |
| NFR-PERF-03 | Performance | Should | Standard profile sustains 10 readings per second for 15 minutes. | Run 50 sensors at 5-second intervals on 16 GB RAM. |
| NFR-SCALE-01 | Scalability | Must | Lite profile supports 10 sensors in 5 cities on 8 GB RAM. | Trainer pre-check before week 1. |
| NFR-SCALE-02 | Scalability | Should | Standard profile supports 50 sensors in 10 cities on 16 GB RAM. | Standard profile demo. |
| NFR-REL-01 | Reliability | Must | Restart causes no data loss from committed Kafka offsets. | Restart test with checkpoint preserved. |
| NFR-REL-02 | Reliability | Must | Delta and PostgreSQL have no duplicate business keys after replay. | Failure-after-sink-write recovery test. |
| NFR-SEC-01 | Security | Must | No secrets appear in source, logs, or Git history. | Gitleaks and manual PR review. |
| NFR-SEC-02 | Security | Must | All published ports bind to `127.0.0.1` using doc 06 mappings; internal broker/controller/driver ports are not published. | Compose review and TC-SEC-002. |
| NFR-SEC-03 | Security | Must | Inspection uses a SELECT-only user for gold/alert serving tables, not raw data or writes. CLI filters use validated, parameterized SQL. | TC-CLI-005. |
| NFR-PRIV-01 | Privacy | Must | No real personal data is stored. | Test data and station data review. |
| NFR-MAINT-01 | Maintainability | Must | Ruff and mypy pass on application packages. | CI lint and type-check jobs. |
| NFR-MAINT-02 | Maintainability | Must | Pure transformation functions support batch unit tests. | Unit tests create small Spark DataFrames. |
| NFR-OBS-01 | Observability | Must | Each micro-batch logs query name, batch ID, input count, output count, and error count. | Log inspection in deterministic test. |
| NFR-OBS-02 | Observability | Should | Consumer lag and streaming progress appear in logs. | Standard profile operations demo. |
| NFR-USE-01 | Usability | Must | Doc 06's single start/stop interface works from a clean clone after initial downloads in 10 steps or fewer. | TC-LOCAL-001 and trainer pre-check. |
| NFR-USE-02 | CLI contract clarity | Must | JSON/text/CSV inspection preserves exact columns, decimal values, PROVISIONAL/FINAL status, CPCB category labels, and `15-minute indicative` basis. Null AQI is not zero; text says `Insufficient data`. | TC-E2E-004 and TC-CLI-001 through TC-CLI-004. |
| NFR-PORT-01 | Portability and persistence | Must | Lite runs on Windows 11 WSL2 Ubuntu, macOS, and Linux offline after downloads, with fictional seeds, fixed loopback ports, persistent stop/start data, and clear failures. | TC-LOCAL-001 through TC-LOCAL-004 and documented platform pre-check. |
| NFR-LIC-01 | Licence compliance | Must | Mandatory tools allow free local educational use. | Version and licence table in document 06. |
| NFR-TEST-01 | Test coverage | Must | Line coverage is at least 80%; branch coverage is at least 70%. | CI coverage report. |
| NFR-TEST-02 | AQI test coverage | Must | AQI module branch coverage is 100%. | Boundary-value test report. |

## Hardware profiles

| Profile | RAM | Services | Proposed limits, not measurements |
|---|---:|---|---|
| Lite | 8 GB, 4 cores | One Kafka broker, one shared Spark driver, PostgreSQL, simulator/poller; short-lived CLI | Kafka 768 MiB, Spark 2048 MiB including Python workers with 1280m heap, PostgreSQL 256 MiB, simulator/poller 192 MiB combined, CLI 128 MiB; total 3392 MiB; no UI services. |
| Standard | 16 GB recommended | Same core services; optional Mailpit | 50 sensors, 10 cities, full pollutants, opt-in Open-Meteo; proposed budgets in doc 06. Built-in consoles are separate optional operations inspection only. |

Windows users SHOULD use 4 GB WSL memory and 4 GB swap on an 8 GB host, or 8 GB WSL memory and 4 GB swap on a 16 GB host. The trainer MUST measure actual peak RAM/swap during the implemented lite pre-check, including non-heap/Python memory; this spec does not claim a measured fit.

## Performance requirements

| Scenario | Required result | Evidence |
|---|---|---|
| Lite steady run | 10 sensors publish for 15 minutes without query failure. | Health check and log summary. |
| End-to-end latency | p95 from Kafka ingest to queryable PROVISIONAL PostgreSQL results is 120 seconds or less. | CLI/SQL timestamp comparison report. |
| Micro-batch stability | 95% of micro-batches complete in 30 seconds or less in lite mode. | Structured Streaming progress logs. |
| Deterministic test | 100 events produce Bronze 100, Silver 87, DLQ 6, late dropped 2, duplicate dropped 5. | Automated E2E report. |

## Reliability requirements

VayuStream MUST use checkpoints for every streaming query. Delta and PostgreSQL sinks MUST use keys or transaction patterns that make replay safe. Kafka and alert consumers MUST expect duplicate delivery and deduplicate by stable IDs.

| Failure | Target | Verification |
|---|---|---|
| Spark query restart | No committed Kafka data is lost. | Kill and restart Silver query. |
| Sink write replay | No duplicate PostgreSQL gold rows. | Inject failure after sink write before checkpoint commit. |
| Invalid input burst | Valid events continue to Silver while invalid events enter quarantine. | Fault-injection test. |
| Serving database outage | Pipeline logs sink failure and health becomes degraded; inspection fails clearly without partial output. | Stop PostgreSQL during an isolated demo run and run TC-LOCAL-004. |

## Security and privacy requirements

The project stores environmental readings and station metadata. It does not need personal data. DPDP Act 2023 awareness is still required because students must explain why personal data is not collected.

| Topic | Requirement | Verification |
|---|---|---|
| Secrets | Use `.env.example` only; never commit real passwords or API keys. | Gitleaks and PR checklist. |
| Network exposure | Bind published services to `127.0.0.1`; Docker-internal bridge networking is separate and not published. | Compose review. |
| OWASP awareness | Validate CLI filters and parameterize inspection/operations queries. | TC-CLI-004, static checks, and code review. |
| Dependency risk | Run pip-audit and Trivy in CI. | CI security job. |
| Logs | Logs MUST NOT include raw secrets or full environment dumps. | Log review in test run. |

## Maintainability and observability requirements

Application packages MUST pass Ruff and mypy. Tests SHOULD avoid sleeping when deterministic event-time control can advance watermarks. Logs MUST be structured JSON inside containers. Each record SHOULD include `query_name`, `batch_id`, `event_id` or `alert_id` when applicable, and `correlation_id` for run-level tracing.

## CLI usability requirements

The README MUST start the lite system in 10 steps or fewer after initial downloads. CLI output is deterministic JSON/text/CSV, not a terminal dashboard. Current AQI includes category, indicative basis, and window status; text uses `Insufficient data` for null AQI. Logs/errors go to stderr; an empty successful result is distinct from an unavailable dependency.

## Portability and licence requirements

The mandatory stack MUST run locally on Windows 11 WSL2 Ubuntu, macOS, and Linux after initial downloads, without internet, paid accounts, API keys, public hostnames, or cloud resources. Cloud-deployment exercises and student frontend work are not permitted at any priority. Mandatory tool choices MUST be free for local educational use.

[Back to README](../README.md)
