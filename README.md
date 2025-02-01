# Vetty Assingment Questions

###2. How many stores receive at least 5 orders/transactions in October 2020?
## SQL Query: Order Count Per Store for October 2020

The following query retrieves the number of orders (`order_count`) per `store_id` from the `transactions` table within October 2020. It only includes stores with at least 5 orders.

```sql
SELECT
    store_id,
    COUNT(*) AS order_count
FROM
    transactions
WHERE
    purchase_time >= '2020-10-01 00:00:00'
    AND purchase_time < '2020-11-01 00:00:00'
GROUP BY
    store_id
HAVING
    COUNT(*) >= 5;

###3. For each store, what is the shortest interval (in minutes) from purchase to refund time?

The following query calculates the shortest interval (in minutes) between the `purchase_time` and `refund_item` for each store where a refund occurred.

```sql
SELECT
    store_id,
    MIN(TIMESTAMPDIFF(MINUTE, purchase_time, refund_item)) AS shortest_interval_in_min
FROM
    transactions
WHERE
    refund_item IS NOT NULL
GROUP BY
    store_id;


