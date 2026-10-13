# Project overview

## Problem

A junior data engineer needs to turn a small, understandable set of retail orders and product records into trustworthy analytics data. This project teaches the dbt workflow from declared source to staging, tests, and a sales mart without requiring a warehouse account or a large platform.

## Outcome

In five days and exactly 30 hours, implement one local dbt project that loads the fictional fixture, creates `stg_orders` and `stg_products`, and produces `mart_sales`. The mart has one row per order, exactly 20 rows for the supplied fixture, and a net-sales total of `441.00`.

## Scope

**In scope**

- Python 3.12, PostgreSQL 18, dbt Core 1.12.x, dbt-postgres 1.11.x, and Docker Compose.
- One dbt source group named `retail`, containing the seeded `orders` and `products` tables.
- Two staging models and one full-refresh sales mart.
- One optional intermediate model only when it fits inside the same 30-hour plan.
- dbt key, relationship, accepted-values, and singular business tests.
- Deterministic local operation after dependencies and the database image are downloaded.

**Out of scope**

- Spark, Kafka, Delta Lake, Airflow, streaming, sensors, AQI, or hardware.
- Charts, dashboards, Grafana, a browser interface, a custom API, or a hosted documentation site.
- Cloud, external databases, live APIs, or any external runtime service.
- More than 100 fictional orders or 20 fictional products.
- A second database, an orchestration platform, or a generalized analytics framework.

## Success measures

| Measure | Required result |
|---|---|
| Source fixture | 20 orders and 6 products; within the 100-order/20-product caps |
| Required models | `stg_orders`, `stg_products`, `mart_sales` |
| Mart grain | One row per `order_id` |
| Demo mart | 20 rows; `SUM(net_sales) = 441.00` |
| Local workflow | Seed, full-refresh build, and test run on local PostgreSQL |
| Time | All Must work, tests, setup, and demo fit exactly within 5 × 6 hours |

## Skills mapped to junior data-engineering work

| Skill | Job task demonstrated |
|---|---|
| dbt sources and seeds | Make input tables and their ownership explicit |
| SQL staging | Cast source fields and preserve source grain |
| Relational modeling | Join a fact-like order mart to a product lookup |
| Data quality | Detect duplicate, null, orphan, invalid-status, and invalid-business rows |
| Reproducibility | Rebuild the same local result from a clean schema |
| PostgreSQL operations | Connect locally, seed data, and report an unavailable database clearly |

[Back to README](../README.md)
