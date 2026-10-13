# Five-day plan and deliverables

The project is exactly five days of six hours each: **30 hours total**. Setup, dbt learning, tests, clean run, and demo all fit inside this allocation.

| Day | Hours | Goal and work | Evidence |
|---|---:|---|---|
| Day 1 | 6 | Read the brief; prepare Python 3.12, uv, Docker Compose, and PostgreSQL; inspect the fixture and grains; run `dbt parse`; build one thin slice and one test. | Local target connects; source and model plan are clear. |
| Day 2 | 6 | Load the two fixture tables with dbt seeds; declare the `retail` source; implement `stg_orders` and `stg_products`; add key tests. | 20 order rows and 6 product rows; staging models retain their grains. |
| Day 3 | 6 | Implement the full-refresh `mart_sales` join and calculation; verify row count and total; use only remaining time within this day for the optional intermediate model. | 20 mart rows and `441.00` net sales. |
| Day 4 | 6 | Run all dbt tests; inject duplicate/null keys, orphan product, invalid status, and business-rule defect in isolated data; repeat the full-refresh build; complete Python coverage if Python helpers exist. | All clean tests pass; each injected defect fails the intended test; 80% line/70% branch target for authored Python helpers. |
| Day 5 | 6 | Run the demo from a clean schema with external network blocked; fix defects; finish README/test notes; rehearse the five-minute demo and explain the data grains. | Clean seed/build/test evidence, exact results, and final explanation. |
| **Total** | **30** | **Five days × six hours** | |

## Deliverables

- One local dbt project using the documented version families.
- One `retail` source group, two required staging models, and one required sales mart.
- The exact fixture results: 20 mart rows and `441.00` in net sales.
- The nine required dbt test nodes and 18 acceptance cases from doc 09.
- A clean local demo after downloads, with a short description of how to run it.

## Five-minute demo

| Minute | Show | Explain |
|---:|---|---|
| 0–1 | Local command sequence and PostgreSQL binding | One local database; offline after downloads |
| 1–2 | Seed/source and staging results | Source grain, casts, 20 orders, and 6 products |
| 2–3 | `mart_sales` result | One row per order, 20 rows, and `441.00` |
| 3–4 | dbt test output | Key, relationship, accepted-values, and singular business checks |
| 4–5 | Repeat build and one failure case | Full-refresh repeatability and a specific failing test |

## Optional practice boundary

Only after every Must item is complete, and only within unused time in the same 30 hours, the learner may add `int_order_sales` and explore one incremental-model version. It is ungraded, not required for the demo, and cannot extend the schedule or displace required tests.

[Back to README](../README.md)
