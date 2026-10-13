# Testing strategy and test cases

## Strategy and gates

Use dbt parse/build/test and isolated local PostgreSQL data. Failure-injection cases must not alter the required committed fixture. Restore the clean test database before the demo. All runtime cases use fictional data and pass without external network access after initial downloads.

There are exactly 19 unique acceptance cases below. The clean project defines nine dbt test nodes as listed in doc 08. Any authored Python support package targets at least **80% line coverage and 70% branch coverage**; do not add Python application logic just to create a coverage denominator. dbt SQL is evaluated with the named dbt tests and exact-result cases.

| ID | Priority | Input and action | Expected result | FR |
|---|---|---|---|---|
| TC-DBT-001 | Must | Run `dbt parse` with the local target and valid project configuration. | Parse succeeds; the graph has one `retail` source group, two required staging models, and `mart_sales`. | FR-DBT-01, FR-DBT-02, FR-DBT-03 |
| TC-DBT-002 | Must | Start with an empty local schema; run `dbt parse`, seed, full-refresh build, and test. | Required source tables and models are created; the clean run succeeds from empty schema. | FR-DBT-01, FR-DBT-02, FR-DBT-03, FR-DBT-04 |
| TC-DBT-003 | Must | Inspect seeded fixture counts and compare with project caps. | `orders` has 20 rows, `products` has 6 rows; neither exceeds 100/20. | FR-DBT-01 |
| TC-DBT-004 | Must | Build `stg_orders` from the clean fixture. | 20 rows; dates, integer quantities, exact two-decimal prices, and status values use the documented types/representations. | FR-DBT-02 |
| TC-DBT-005 | Must | Build `stg_products` from the clean fixture. | 6 rows; product keys, names, and categories are present at product grain. | FR-DBT-02 |
| TC-DBT-006 | Must | Inspect the clean sales mart by order key. | Exactly 20 rows; `order_id` is unique and non-null. | FR-DBT-03, FR-DBT-04 |
| TC-DBT-007 | Must | Sum `mart_sales.net_sales` on the clean fixture. | Exact numeric total is `441.00`; 18 orders are `COMPLETE`, 2 are `CANCELLED`. | FR-DBT-03 |
| TC-DBT-008 | Must | Run `dbt test` on the clean fixture and inspect the test results. | All nine required nodes pass: unique/not-null on the three keys, one relationship, one accepted-values rule, and one singular business check. | FR-DBT-04 |
| TC-DBT-009 | Must | In an isolated `orders` seed, duplicate one `order_id`; rebuild and test. | The `stg_orders.order_id` unique test fails and reports the duplicate key. | FR-DBT-04 |
| TC-DBT-010 | Must | In an isolated `orders` seed, set one `order_id` to null; rebuild and test. | The `stg_orders.order_id` not-null test fails. | FR-DBT-04 |
| TC-DBT-011 | Must | In an isolated `products` seed, duplicate one `product_id`; rebuild and test. | The `stg_products.product_id` unique test fails. | FR-DBT-04 |
| TC-DBT-012 | Must | In an isolated `products` seed, set one `product_id` to null; rebuild and test. | The `stg_products.product_id` not-null test fails. | FR-DBT-04 |
| TC-DBT-013 | Must | Set an order's `product_id` to a key absent from `products`; rebuild and test. | The single mart-to-product relationship test fails and identifies the orphan key. | FR-DBT-04 |
| TC-DBT-014 | Must | Set an order status to `PENDING`; rebuild and test. | The accepted-values test fails; only `COMPLETE` and `CANCELLED` are accepted. | FR-DBT-04 |
| TC-DBT-015 | Must | In an isolated test database, build the mart, corrupt one completed row's `net_sales`, and run `dbt test`. | The singular business test fails and returns the invalid row. | FR-DBT-04 |
| TC-DBT-016 | Must | Run the same seed and full-refresh build twice, checking after each run. | Both runs leave 20 mart rows and total `441.00`; no duplicate mart rows accumulate. | FR-DBT-03, FR-DBT-05 |
| TC-DBT-017 | Must | Stop PostgreSQL, then run a command that needs the database. | dbt exits nonzero with a useful connection error; no fallback source or false success occurs. | FR-DBT-01, FR-DBT-05 |
| TC-DBT-018 | Must | With dependencies and the image already downloaded, block external network and run the complete demo. | Compose database, seed, build, and tests finish using only local resources. | FR-DBT-05 |
| TC-DBT-019 | Must | Complete the demo, stop and restart PostgreSQL without deleting its volume, then inspect the mart. | Existing local state remains at 20 rows and `441.00`; a repeat full-refresh build still does not accumulate duplicates. | FR-DBT-05 |

## Coverage and evidence

For any project-owned Python helper package, report line and branch coverage separately and meet the 80%/70% thresholds. SQL transformations do not count as Python coverage; their evidence is the model-test results, `20` mart rows, and `441.00` total. CI and the trainer review the test outputs and exact result table, not an application screenshot.

[Back to README](../README.md)
