# Evaluation rubric

All grading gates are achievable within five days and 30 hours. There are no bonus points; optional practice is ungraded.

## Mandatory gates

- `dbt parse`, seed, full-refresh build, and `dbt test` complete from a clean local schema.
- The implementation has one `retail` source group, two required staging models, and one sales mart.
- The clean fixture produces exactly 20 mart rows and `441.00` in net sales.
- The nine required dbt test nodes pass on clean data; the five specified defect types fail their corresponding test.
- The same-file full-refresh rerun preserves row count and total.
- The demo runs locally after downloads without an external runtime service.
- Any authored Python support package meets 80% line and 70% branch coverage; SQL/dbt is covered by its dbt tests and exact-result cases.
- The learner can explain source grain, mart grain, joins, business rule, and test results.

## Weighted rubric

| Category | Points |
|---|---:|
| Functional and data correctness | 40 |
| Data modeling, SQL, and ETL practice | 25 |
| Tests and CI | 15 |
| Local reproducibility | 10 |
| Documentation and Git | 5 |
| Demo and viva | 5 |
| **Total** | **100** |

## Scoring guidance

| Category | Full-credit evidence |
|---|---|
| Functional and data correctness | Required source/staging/mart graph; correct order grain; exact 20 rows and `441.00`; repeatable full refresh |
| Data modeling, SQL, and ETL practice | Correct data types, clear grains, stable keys, product relationship, and correct cancelled-order treatment |
| Tests and CI | Nine dbt test nodes; all 18 acceptance cases accounted for; required CI checks pass |
| Local reproducibility | Compose database on loopback; clean-schema demo succeeds offline after downloads; unavailable database fails clearly |
| Documentation and Git | Concise run instructions, linked spec, clear commits, no secrets or generated state |
| Demo and viva | Five-minute demonstration and accurate explanation of a model/test decision |

The optional intermediate model and incremental practice do not earn points and are not gates.

[Back to README](../README.md)
