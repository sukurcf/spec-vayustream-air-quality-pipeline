# Evaluation rubric

Purpose: This document defines mandatory gates, scoring, bonus rules, deductions, grade bands, and viva questions for VayuStream.

## Mandatory gates

If any gate fails, the result is "Rework required" for any score.

| Gate | Required evidence |
|---|---|
| CI is green on `main` | GitHub Actions shows required checks passing. |
| Coverage thresholds are met | Line coverage is at least 80%; branch coverage is at least 70%; AQI branch coverage is 100%. |
| Clean clone starts offline | After initial downloads, doc 06's single start entry point initializes healthy loopback lite services/fictional seeds; TC-LOCAL-001 passes with one broker, one driver, and no UI services. |
| Local core behavior and persistence | TC-LOCAL-002 through TC-LOCAL-004 prove exact offline counts/inspection, preserved offsets/checkpoints/poller/outbox, and clear input/dependency failures. |
| Meaningful test minimum | At least 45 cases including all four inspection-command snapshots and four local acceptance tests. |
| No secrets in Git history | Gitleaks and manual review find no secrets. |
| Student can explain code | Viva answers show ownership of Kafka, Spark, AQI, and SQL code. |
| Must scope works | All Must FRs from FR-SIM-01 through FR-TEST-01 work in the demo. |
| Honest delivery semantics | Kafka and notifications are described as at-least-once, not exactly-once. |

## Weighted rubric

| Category | Points |
|---|---:|
| Functional completeness (Must FRs work in the demo) | 30 |
| Code quality and design | 15 |
| Testing and coverage | 15 |
| DevOps: CI/CD, Docker, reproducibility | 10 |
| Documentation (README, ADRs, API/CLI/pipeline docs) | 10 |
| Git and engineering practices (PRs, commits, issues) | 5 |
| Final demo and viva | 15 |
| **Total** | **100** |

Version 1.1 grading impact: frontend and cloud-deployment deliverables are removed at every priority. Their effort and points are reassessed through Python inspection, SQL/data fidelity, tests, offline local operation, and recovery evidence within the unchanged weights. Built-in Kafka/Grafana consoles are optional operations tools, never graded business deliverables.

## Project-specific criteria

| Category | Excellent | Good | Needs work |
|---|---|---|---|
| Functional completeness | All Must FRs work; 100-message counts remain Bronze 100, Silver 87, DLQ/quarantine 6, late 2, duplicates 5; four inspection commands preserve exact data/null/status semantics; alerts/outbox/recovery are correct. | Most Must FRs work; one minor CLI-help or runbook issue does not affect correctness. | Any core Kafka-to-serving-table/CLI path fails, or Must FR evidence is missing. |
| Code quality and design | Pure transformations; Kafka, shared Spark driver, AQI, sinks, alert poller, inspection serialization, and ops are separated. | Main modules are understandable, but some Spark logic is too tightly coupled to I/O. | Mixed scripts hide boundaries, CLI recomputes AQI, or separate per-query drivers overload lite. |
| Testing and coverage | Meets separate line/branch and AQI gates; at least 45 tests include boundaries, exact format snapshots, deterministic batches, recovery, and local acceptance. | Meets general coverage gates; one advanced recovery test is manual but documented. | Coverage is below threshold, flaky, or misses AQI/watermark, offline operation, or persistence. |
| DevOps: CI/CD, Docker, reproducibility | Offline clean start after image/JAR downloads; loopback ports, one driver, and persistent storage verified; pinned images/scans; actual RAM evidence compared with proposed budgets. | CI and Compose work, but setup needs minor manual correction documented in README. | CI incomplete, `latest` images, runtime Maven/API downloads, hidden state, data loss on stop, or unmeasured resource claims. |
| Documentation | README, runbook, ADRs, CHANGELOG, and exact inspection/local contracts match behavior; explicit simulator versus optional real source is clear. | Docs are mostly correct, with small gaps around optional Should features. | Docs are generic, stale, or omit replay, checkpoints, and delivery semantics. |
| Git and engineering practices | Small PRs, clear issues, Conventional Commits, and useful review responses show steady work. | Most work uses PRs, but some commits are broad or messages are weak. | Large unreviewed commits or direct pushes hide the development process. |
| Final demo and viva | Demo fits 10 minutes; student explains Kafka offsets, Spark watermarks, Delta, AQI, and upserts without guessing. | Demo is clear, but one deep topic needs trainer prompting. | Student cannot explain key code or cannot run the lite profile live. |

## Bonus rules

Bonus is up to +10 points. Bonus applies only when the base score is 60 or more and all mandatory gates pass. The final score is capped at 100.

| Bonus item | Maximum points | Conditions |
|---|---:|---|
| Avro with open-source schema registry | 3 | Could item works locally and ADR explains 8 GB impact. |
| Maintained object storage experiment | 3 | Could item works locally and ADR verifies current licence and maintenance status. |
| Sensor-drift anomaly detection | 2 | Could z-score drift feature works and does not affect Must AQI results. |
| Python replay-verification evidence bundle | 2 | Could command compares saved offsets and key counts before/after a controlled sink-write crash/replay, proving zero duplicate keys without changing Must data or adding UI/cloud services. |

## Deductions

| Issue | Deduction |
|---|---:|
| Claims official real-time AQI instead of indicative 15-minute AQI | -5 |
| Promises exactly-once e-mail, webhook, or Kafka notification delivery | -5 |
| Uses real personal data or real sensor credentials | Rework required |
| Skips the 8 GB lite profile | Rework required |
| Uses unpinned Docker image tags | -5 |
| Missing ADRs for major alternatives | -3 each |
| CLI omits indicative/category/window-status labels or replaces null AQI with zero | -3 |
| Runbook cannot replay offsets or reprocess Bronze to Silver | -5 |
| Demo video exceeds 5 minutes | -2 |
| Significant AI use is hidden in PR descriptions | -5 |

## Grade bands

| Score | Result |
|---:|---|
| 85-100 | Distinction |
| 70-84 | Merit |
| 60-69 | Pass |
| Below 60 | Rework required |

## Viva questions

| # | Question | Strong answer includes |
|---:|---|---|
| 1 | Why is `station_id` the Kafka key? | Per-station ordering; partitioning; replay by station; joins with station reference. |
| 2 | What is stored in Bronze that Silver does not store? | Raw payload; Kafka topic, partition, offset; audit value; duplicate raw events. |
| 3 | Why do valid late events not go to DLQ? | Schema is valid; watermark makes them too late; count in metrics; avoid false data-quality errors. |
| 4 | How does the 15-minute watermark affect duplicates? | State kept for event IDs; duplicates within watermark dropped; very late valid events dropped. |
| 5 | Why is the city AQI called indicative? | Uses 15-minute averages; official method uses longer averages; label prevents misleading claims. |
| 6 | How do you avoid duplicate PostgreSQL rows after replay? | Business key; idempotent upsert; foreachBatch; restart test after sink write. |
| 7 | What is at-least-once delivery in this project? | Kafka or notification may repeat; stable `event_id` and `alert_id`; consumers deduplicate. |
| 8 | What does completeness measure? | Unique valid readings divided by expected readings; 15-minute window; active sensors and 5-second interval. |
| 9 | Why does lite use one shared driver and no UI services? | 8 GB/four-core target, 4 GB WSL allocation, all-process proposed RAM budget, Python/non-heap overhead, 10 stations, and measured trainer pre-check rather than assumed fit. |
| 10 | How would you replay data safely from Kafka offsets? | Stop query; preserve or reset checkpoint intentionally; choose offsets; write idempotently; verify counts. |

[Back to README](../README.md)
