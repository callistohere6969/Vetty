# Vetty Assingment Questions

2. How many stores receive at least 5 orders/transactions in October 2020?
 

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
```

3. For each store, what is the shortest interval (in minutes) from purchase to refund time?

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
```
4. What is the gross_transaction_value of every store’s first order?

```sql
SELECT
    t.store_id,
    t.gross_transaction_value,
    t.purchase_time
FROM
    transactions t
INNER JOIN (
    SELECT
        store_id,
        MIN(purchase_time) AS first_time
    FROM
        transactions
    GROUP BY
        store_id
) AS first_orders 
ON
    t.store_id = first_orders.store_id
    AND t.purchase_time = first_orders.first_time;

```
5. What is the most popular item name that buyers order on their first purchase?
```sql
SELECT
    i.item_name,
    COUNT(*) AS order_count
FROM
    transactions t
INNER JOIN items i
    ON t.store_id = i.store_id AND t.item_id = i.item_id
WHERE
    (t.buyer_id, t.purchase_time) IN (
        SELECT
            buyer_id,
            MIN(purchase_time) AS first_purchase_time
        FROM
            transactions
        GROUP BY
            buyer_id
    )
GROUP BY
    i.item_name
ORDER BY
    order_count DESC
LIMIT 1;
```

6. Create a flag in the transaction items table indicating whether the refund can be processed or
not. The condition for a refund to be processed is that it has to happen within 72 of Purchase
time.

```sql
SELECT
    *,
    CASE
        WHEN refund_item IS NOT NULL AND TIMESTAMPDIFF(HOUR, purchase_time, refund_item) <= 72 THEN 'Yes'
        ELSE 'No'
    END AS refund_processable
FROM
    transactions;
```

