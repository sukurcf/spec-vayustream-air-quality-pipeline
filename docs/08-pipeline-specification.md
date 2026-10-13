# dbt pipeline specification

## Model graph and materialization

| Name | Type | Required behavior |
|---|---|---|
| `retail.orders` | Source table loaded by dbt seed | 20 fixture rows; one row per order |
| `retail.products` | Source table loaded by dbt seed | 6 fixture rows; one row per product |
| `stg_orders` | Staging model | Cast date, quantity, and unit price; retain order grain and status |
| `stg_products` | Staging model | Expose product key, name, and category; retain product grain |
| `mart_sales` | Sales mart | Join order to product; one row per order; calculate `net_sales` |
| `int_order_sales` | Optional intermediate model | May separate the join/calculation only if it fits in the 30 hours |

Required models use repeatable full-refresh behavior. The optional intermediate model is also full-refresh by default. A single incremental-model experiment is allowed only as ungraded practice and only if it fits in the same plan; it does not change the required full-refresh demo.

## Required commands

Run in order from the student implementation repository:

```text
docker compose up -d db
uv run dbt parse --target local
uv run dbt seed --target local
uv run dbt build --target local --full-refresh
uv run dbt test --target local
```

`dbt seed` materializes the two small source tables in local PostgreSQL. `dbt build` creates the required models and runs their selected tests; the separate `dbt test` command provides explicit test evidence. A successful full-refresh build is safe to repeat.

## Mart contract

`mart_sales` has one row for each `order_id` and includes: `order_id`, `order_date`, `product_id`, `product_name`, `category`, `quantity`, `unit_price`, `status`, and `net_sales`.

| Expected result | Value |
|---|---:|
| Seeded `orders` rows | 20 |
| Seeded `products` rows | 6 |
| `stg_orders` rows | 20 |
| `stg_products` rows | 6 |
| `mart_sales` rows | 20 |
| Complete orders | 18 |
| Cancelled orders | 2 |
| `SUM(mart_sales.net_sales)` | `441.00` |

The singular business test returns a failing record if `net_sales` is not quantity times unit price for `COMPLETE`, or is not zero for `CANCELLED`.

## Test contract

The clean-fixture project defines nine required dbt test nodes:

1. Unique and not-null tests for `stg_orders.order_id`.
2. Unique and not-null tests for `stg_products.product_id`.
3. Unique and not-null tests for `mart_sales.order_id`.
4. One relationship test from `mart_sales.product_id` to `stg_products.product_id`.
5. One accepted-values test for `stg_orders.status` (`COMPLETE`, `CANCELLED`).
6. One singular `net_sales` business check.

The clean fixture passes all nine. An injected duplicate/null key, orphan product key, unsupported order status, or incorrect business value makes its relevant dbt test fail.

## Failure and local-operation behavior

If PostgreSQL is unavailable, the relevant command fails nonzero and reports a connection problem. Do not fall back to SQLite, a hosted database, or a live source. After initial downloads, the commands run without external network access. Stop the database with `docker compose down`; retain the local volume unless the learner deliberately resets their local test environment.

dbt artifacts may be generated and inspected locally. Serving or styling a documentation website is outside the project.

[Back to README](../README.md)
