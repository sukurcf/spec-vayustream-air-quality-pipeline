# Interview preparation

## One-minute explanation

“I built a small local dbt project for fictional retail data. A single `retail` source group contains orders and products. Two staging models cast and preserve each source grain; a full-refresh sales mart joins products to order rows and computes net sales. The supplied fixture produces 20 mart rows and `441.00`. dbt tests cover keys, product relationships, valid statuses, and the sales rule. I verified the build from an empty local PostgreSQL schema, repeated it, and tested injected data defects without using a cloud warehouse or live API.”

## Ten role-specific questions

| # | Question | Key points for a strong answer |
|---:|---|---|
| 1 | What does the `retail` source represent? | One dbt source group with two locally seeded tables; external inputs and dbt models are distinct concepts. |
| 2 | Why create staging models before a mart? | Typed, stable names; preserve input grain; keep source cleanup separate from business logic. |
| 3 | What is the grain of `mart_sales`? | One row per `order_id`; each fixture order has one product. |
| 4 | Why use a relationship test for `product_id`? | Proves every mart product key resolves to the product staging key and catches orphans. |
| 5 | What does the accepted-values test protect? | Restricts order status to `COMPLETE` or `CANCELLED`. |
| 6 | What does the singular business test verify? | `net_sales` equals quantity × unit price for completed orders and zero for cancelled orders. |
| 7 | Why are full-refresh models the default here? | The fixture is small; rebuilds are easy to reason about and repeat; incremental state is unnecessary for the required outcome. |
| 8 | How do you know the join did not multiply orders? | Compare the one-row-per-order grain, mart unique/not-null tests, and the exact 20-row result. |
| 9 | How did you prove repeatability and offline operation? | Repeated the same full-refresh demo; blocked external network after preparing packages and image; compared 20 rows and `441.00`. |
| 10 | When would an incremental model be useful, and what does it add? | Larger changing data can avoid rebuilding all rows; it adds state, unique-key, late-change, and full-refresh recovery concerns, so it is optional here. |

[Back to README](../README.md)
