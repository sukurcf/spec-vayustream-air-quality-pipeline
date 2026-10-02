# Non-functional requirements

Purpose: This document defines measurable quality targets for VayuStream and how the student verifies them.

## NFR catalog

| ID | Category | Priority | Target | Verification method |
|---|---|---|---|---|
| NFR-PERF-01 | Performance | Must | Lite p95 ingest-to-dashboard latency is 120 seconds or less. | Run 10 sensors for 15 minutes and compare Kafka timestamp with PostgreSQL visible time. |
| NFR-PERF-02 | Performance | Must | Producer sustains 2 readings per second for 10 minutes with zero unhandled send errors. | Load test in lite profile. |
| NFR-PERF-03 | Performance | Should | Standard profile sustains 10 readings per second for 15 minutes. | Run 50 sensors at 5-second intervals on 16 GB RAM. |
| NFR-SCALE-01 | Scalability | Must | Lite profile supports 10 sensors in 5 cities on 8 GB RAM. | Trainer pre-check before week 1. |
| NFR-SCALE-02 | Scalability | Should | Standard profile supports 50 sensors in 10 cities on 16 GB RAM. | Standard profile demo. |
| NFR-REL-01 | Reliability | Must | Restart causes no data loss from committed Kafka offsets. | Restart test with checkpoint preserved. |
| NFR-REL-02 | Reliability | Must | Delta and PostgreSQL have no duplicate business keys after replay. | Failure-after-sink-write recovery test. |
| NFR-SEC-01 | Security | Must | No secrets appear in source, logs, or Git history. | Gitleaks and manual PR review. |
| NFR-SEC-02 | Security | Must | Containers expose only local development ports. | Compose review and local port scan. |
| NFR-PRIV-01 | Privacy | Must | No real personal data is stored. | Test data and station data review. |
| NFR-MAINT-01 | Maintainability | Must | Ruff and mypy pass on application packages. | CI lint and type-check jobs. |
| NFR-MAINT-02 | Maintainability | Must | Pure transformation functions support batch unit tests. | Unit tests create small Spark DataFrames. |
| NFR-OBS-01 | Observability | Must | Each micro-batch logs query name, batch ID, input count, output count, and error count. | Log inspection in deterministic test. |
| NFR-OBS-02 | Observability | Should | Consumer lag and streaming progress appear in logs. | Standard profile operations demo. |
| NFR-USE-01 | Usability | Must | Lite system starts from clean clone in 10 steps or fewer. | README setup trial by another student. |
| NFR-ACC-01 | Accessibility | Must | Grafana panels use CPCB labels in addition to colours. | Dashboard review with screenshots. |
| NFR-PORT-01 | Portability | Must | Lite profile runs on Windows 11 WSL2, macOS, and Linux with Docker. | Documented platform smoke checks. |
| NFR-LIC-01 | Licence compliance | Must | Mandatory tools allow free local educational use. | Version and licence table in document 06. |
| NFR-TEST-01 | Test coverage | Must | Line coverage is at least 80%; branch coverage is at least 70%. | CI coverage report. |
| NFR-TEST-02 | AQI test coverage | Must | AQI module branch coverage is 100%. | Boundary-value test report. |

## Hardware profiles

| Profile | RAM | Services | Limits |
|---|---:|---|---|
| Lite | 8 GB | Kafka 1 broker, Spark local mode, PostgreSQL, Grafana, simulator | Kafka 768 MiB, Spark 2304 MiB with 1500m heap, PostgreSQL 384 MiB, Grafana 192 MiB, simulator 128 MiB, no Kafka UI. |
| Standard | 16 GB | Lite services plus optional Kafka UI and Mailpit | 50 sensors, 10 cities, standard dashboard, optional Open-Meteo polling. |

Windows users SHOULD configure WSL2 memory. For 8 GB laptops, use 4 GB WSL memory and 4 GB swap. For 16 GB laptops, use 8 GB WSL memory and 4 GB swap. The trainer MUST validate the lite profile on an 8 GB laptop before week 1.

## Performance requirements

| Scenario | Required result | Evidence |
|---|---|---|
| Lite steady run | 10 sensors publish for 15 minutes without query failure. | Health check and log summary. |
| End-to-end latency | p95 from Kafka ingest to Grafana data availability is 120 seconds or less. | Timestamp comparison report. |
| Micro-batch stability | 95% of micro-batches complete in 30 seconds or less in lite mode. | Structured Streaming progress logs. |
| Deterministic test | 100 events produce Bronze 100, Silver 87, DLQ 6, late dropped 2, duplicate dropped 5. | Automated E2E report. |

## Reliability requirements

VayuStream MUST use checkpoints for every streaming query. Delta and PostgreSQL sinks MUST use keys or transaction patterns that make replay safe. Kafka and alert consumers MUST expect duplicate delivery and deduplicate by stable IDs.

| Failure | Target | Verification |
|---|---|---|
| Spark query restart | No committed Kafka data is lost. | Kill and restart Silver query. |
| Sink write replay | No duplicate PostgreSQL gold rows. | Inject failure after sink write before checkpoint commit. |
| Invalid input burst | Valid events continue to Silver while invalid events enter quarantine. | Fault-injection test. |
| Dashboard database outage | Pipeline logs sink failure and health becomes degraded. | Stop PostgreSQL during a demo run. |

## Security and privacy requirements

The project stores environmental readings and station metadata. It does not need personal data. DPDP Act 2023 awareness is still required because students must explain why personal data is not collected.

| Topic | Requirement | Verification |
|---|---|---|
| Secrets | Use `.env.example` only; never commit real passwords or API keys. | Gitleaks and PR checklist. |
| Network exposure | Bind development services to localhost unless Docker requires bridge networking. | Compose review. |
| OWASP awareness | Avoid injection in dashboard queries and admin scripts. | Static checks and code review. |
| Dependency risk | Run pip-audit and Trivy in CI. | CI security job. |
| Logs | Logs MUST NOT include raw secrets or full environment dumps. | Log review in test run. |

## Maintainability and observability requirements

Application packages MUST pass Ruff and mypy. Tests SHOULD avoid sleeping when deterministic event-time control can advance watermarks. Logs MUST be structured JSON inside containers. Each record SHOULD include `query_name`, `batch_id`, `event_id` or `alert_id` when applicable, and `correlation_id` for run-level tracing.

## Usability and accessibility requirements

The README MUST start the lite system in 10 steps or fewer. Grafana panels MUST not depend on colour alone. Current AQI panels MUST show the CPCB category label and the word `indicative` for 15-minute AQI.

## Portability and licence requirements

The mandatory stack MUST run locally on Windows 11 WSL2, macOS, or Linux. Cloud services are optional only. Every optional cloud experiment MUST include a cost warning and a destroy-after-demo step. Mandatory tool choices MUST be free for local educational use.

[Back to README](../README.md)
