# Functional requirements

The project has exactly five Must requirements. All inputs are fictional and local.

## Must requirements

| ID | Priority | Requirement | User stories |
|---|---|---|---|
| FR-DBT-01 | Must | Load the bounded retail source fixture | US-01, US-03 |
| FR-DBT-02 | Must | Build two grain-preserving staging models | US-01 |
| FR-DBT-03 | Must | Build a repeatable order-grain sales mart | US-02, US-03 |
| FR-DBT-04 | Must | Enforce the required dbt data tests | US-02 |
| FR-DBT-05 | Must | Run and repeat the local demo without external runtime services | US-03 |

### FR-DBT-01 — Load the bounded retail source fixture

The project MUST declare one dbt source group named `retail` with `orders` and `products` tables. `dbt seed` loads the deterministic fixture to local PostgreSQL. The required fixture contains 20 order rows and 6 product rows; the project MUST NOT require more than 100 orders or 20 products.

- Given an empty local schema, when the seed command runs, then `retail.orders` contains 20 rows and `retail.products` contains 6 rows.
- Given either required fixture is missing or the database is unavailable, when seeding starts, then the command fails visibly and does not switch to another source.

### FR-DBT-02 — Build two staging models

The project MUST provide `stg_orders` and `stg_products`. Each model MUST cast its date, integer, and decimal fields to their documented types, retain its source grain, and expose stable column names.

- Given the clean seed fixture, when the models build, then `stg_orders` has 20 rows and `stg_products` has 6 rows.
- Given valid source keys, when staging tests run, then order and product keys are non-null and unique.

### FR-DBT-03 — Build the sales mart

The project MUST build `mart_sales` at one row per `order_id`, joining each order to its product. Required transforms MUST use repeatable full-refresh behavior by default. `net_sales` equals `quantity × unit_price` for `COMPLETE` orders and zero for `CANCELLED` orders.

- Given the clean fixture, when a full-refresh build completes, then `mart_sales` contains exactly 20 order rows.
- Given the clean fixture, when net sales are summed, then the exact total is `441.00`.

### FR-DBT-04 — Enforce the required dbt data tests

The project MUST test unique and not-null order/product/mart keys; exactly one relationship from `mart_sales.product_id` to `stg_products.product_id`; one accepted-values rule for order status; and one singular business check that `net_sales` follows the stated status formula.

- Given clean fixture data, when dbt tests run, then every required test passes.
- Given a duplicate key, null key, orphan product, unsupported status, or incorrect net-sales value is injected in an isolated test dataset, when the affected test runs, then it fails and identifies the failing test.

### FR-DBT-05 — Run a repeatable local demo

The required local path is PostgreSQL via Docker Compose, followed by `dbt seed`, `dbt build`, and `dbt test`. The demo MUST work offline after initial dependency/image downloads and MUST NOT call a live API or remote database.

- Given the demo has already succeeded, when the same seed and full-refresh build run again, then the mart remains at 20 rows and `441.00`.
- Given PostgreSQL is unavailable, when a dbt command requires it, then the command exits nonzero with a useful connection failure; it MUST NOT report a successful build or silently use a different backend.

## Business and data rules

| ID | Rule |
|---|---|
| BR-01 | Source grain is one order per `orders.order_id` and one product per `products.product_id`; the required fixture is 20 orders and 6 products. |
| BR-02 | Only `COMPLETE` and `CANCELLED` are accepted order statuses. |
| BR-03 | `net_sales` is `quantity × unit_price` for `COMPLETE` and `0.00` for `CANCELLED`; money uses exact two-decimal numeric values. |
| BR-04 | Every mart product key must match one product key; an order with no known product is invalid. |
| BR-05 | For the documented demo fixture, mart rows are 20 and total net sales are exactly `441.00`. |

## Optional practice boundary

One intermediate model, `int_order_sales`, may be added only if it fits inside the same five-day/30-hour plan. One model may be explored as incremental as an ungraded practice exercise; the required demo remains a full-refresh run. There is no additional feature catalogue or bonus score.

[Back to README](../README.md)
