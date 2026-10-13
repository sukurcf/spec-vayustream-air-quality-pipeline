# Data model

## Source and grains

The one dbt source group is `retail`; its two tables are loaded from the fictional `orders` and `products` seeds.

| Relation | Grain | Key | Required fields |
|---|---|---|---|
| `retail.orders` | One order, one product per order | `order_id` | `order_date`, `product_id`, `quantity`, `unit_price`, `status` |
| `retail.products` | One product | `product_id` | `product_name`, `category` |
| `stg_orders` | One row per source order | `order_id` | Typed date, integer quantity, exact numeric unit price, status |
| `stg_products` | One row per source product | `product_id` | Product name and category |
| `mart_sales` | One row per order | `order_id` | Order date, product key/name/category, quantity, unit price, status, net sales |
| `int_order_sales` | One row per order | `order_id` | Optional intermediate model; only if it fits the 30-hour plan |

All required transformations preserve the stated grain. The order's `unit_price` is the transaction price. `net_sales` is exact numeric with two decimal places: quantity times unit price for `COMPLETE`, otherwise `0.00` for `CANCELLED`.

## Required dbt tests

| Model column | Test |
|---|---|
| `stg_orders.order_id` | Unique and not null |
| `stg_products.product_id` | Unique and not null |
| `mart_sales.order_id` | Unique and not null |
| `mart_sales.product_id` | One relationship test to `stg_products.product_id` |
| `stg_orders.status` | One accepted-values test: `COMPLETE`, `CANCELLED` |
| `mart_sales.net_sales` | One singular business test for the status formula |

No additional relationship, accepted-values, or singular business test is required.

## Product fixture

| product_id | product_name | category |
|---|---|---|
| P01 | Northstar Mug | Home |
| P02 | Field Notebook | Stationery |
| P03 | Trail Bottle | Outdoor |
| P04 | Cable Set | Electronics |
| P05 | Desk Lamp | Home |
| P06 | Canvas Tote | Accessories |

## Order fixture

`expected_net_sales` below is a documented expected result, not an input column. Each fixture row has one product.

| order_id | order_date | product_id | quantity | unit_price | status | expected_net_sales |
|---|---|---|---:|---:|---|---:|
| O001 | 2026-10-01 | P01 | 2 | 12.50 | COMPLETE | 25.00 |
| O002 | 2026-10-01 | P02 | 1 | 8.00 | COMPLETE | 8.00 |
| O003 | 2026-10-01 | P03 | 1 | 24.00 | COMPLETE | 24.00 |
| O004 | 2026-10-01 | P04 | 3 | 5.50 | COMPLETE | 16.50 |
| O005 | 2026-10-01 | P05 | 1 | 32.00 | COMPLETE | 32.00 |
| O006 | 2026-10-01 | P06 | 2 | 10.00 | COMPLETE | 20.00 |
| O007 | 2026-10-01 | P01 | 1 | 12.50 | COMPLETE | 12.50 |
| O008 | 2026-10-01 | P02 | 3 | 8.00 | COMPLETE | 24.00 |
| O009 | 2026-10-01 | P03 | 2 | 24.00 | COMPLETE | 48.00 |
| O010 | 2026-10-01 | P04 | 1 | 5.50 | COMPLETE | 5.50 |
| O011 | 2026-10-02 | P05 | 2 | 32.00 | COMPLETE | 64.00 |
| O012 | 2026-10-02 | P06 | 1 | 10.00 | COMPLETE | 10.00 |
| O013 | 2026-10-02 | P01 | 4 | 12.50 | COMPLETE | 50.00 |
| O014 | 2026-10-02 | P02 | 2 | 8.00 | COMPLETE | 16.00 |
| O015 | 2026-10-02 | P03 | 1 | 24.00 | CANCELLED | 0.00 |
| O016 | 2026-10-02 | P04 | 2 | 5.50 | COMPLETE | 11.00 |
| O017 | 2026-10-02 | P05 | 1 | 32.00 | COMPLETE | 32.00 |
| O018 | 2026-10-02 | P06 | 3 | 10.00 | COMPLETE | 30.00 |
| O019 | 2026-10-02 | P01 | 1 | 12.50 | COMPLETE | 12.50 |
| O020 | 2026-10-02 | P02 | 1 | 8.00 | CANCELLED | 0.00 |

## Demo contract

The fixture contains 20 orders, 6 products, 18 `COMPLETE` orders, and 2 `CANCELLED` orders. The mart contains 20 order-grain rows. Its exact `SUM(net_sales)` is `441.00`. The permitted project maximum is 100 fictional orders and 20 fictional products; do not expand the required demo beyond that.

[Back to README](../README.md)
