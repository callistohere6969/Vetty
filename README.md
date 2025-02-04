# Vetty Assingment Questions
> The datatype of column 'refund_item' is considered as timestamp for the purpose of analysis



1. What is the count of purchases per month (excluding refunded purchases)?
```sql
SELECT
    DATE_FORMAT(purchase_time, '%Y-%m') AS purchase_month,
    COUNT(*) AS purchase_count
FROM
    transactions
WHERE
    refund_item IS NULL --Excluding refunded purchases
GROUP BY
    DATE_FORMAT(purchase_time, '%Y-%m') --Per Month
ORDER BY
    purchase_month ;
```

2. How many stores receive at least 5 orders/transactions in October 2020?
 

> The following query retrieves the number of orders (`order_count`) per `store_id` from the `transactions` table within October 2020. It only includes stores with at least 5 orders.

```sql
SELECT
    store_id,
    COUNT(*) AS order_count
FROM
    transactions
WHERE
    purchase_time >= '2020-10-01 00:00:00'
    AND purchase_time < '2020-11-01 00:00:00' --Within the month of October
GROUP BY
    store_id
HAVING
    COUNT(*) >= 5; --atleast 5(the value can be changed according to the minimum number of order count required)
```

3. For each store, what is the shortest interval (in minutes) from purchase to refund time?

> The following query calculates the shortest interval (in minutes) between the `purchase_time` and `refund_item` for each store where a refund occurred.

```sql
SELECT
    store_id,
    MIN(TIMESTAMPDIFF(MINUTE, purchase_time, refund_item)) AS shortest_interval_in_min
FROM
    transactions
WHERE
    refund_item IS NOT NULL --considering only the items that has been refunded
GROUP BY
    store_id;
```
> Result is provided only on the basis of the items that has been refunded

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
) AS first_orders --this table contain the first order of each store
ON
    t.store_id = first_orders.store_id
    AND t.purchase_time = first_orders.first_time;
-- joined both the tables (our original table and the 'first_time' table) on the basis of store_id and purchase_time
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
--checking if the buyer exist in the follwing table
    (t.buyer_id, t.purchase_time) IN (
        SELECT
            buyer_id,
            MIN(purchase_time) AS first_purchase_time
        FROM
            transactions
        GROUP BY
            buyer_id
--this table contains the first purchase time of each buyer
    )
GROUP BY
    i.item_name 
ORDER BY
    order_count DESC
LIMIT 1;
```
> Data provided is not that efficient that we can make more observations because there is very few items the transition table that matches the item ID of the item table

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
--checking if the difference in purchase_time and refund_time is less than or equal to 72hours than it would be considered as processed refund else it wouldnt be 
```
> if we want to add the column permanently in the table

```sql
-- Add the refund_processable column
ALTER TABLE transactions
ADD COLUMN refund_processable CHAR(3);

-- Update the column based on the condition
UPDATE transactions
SET refund_processable = 
    CASE
        WHEN refund_item IS NOT NULL AND TIMESTAMPDIFF(HOUR, purchase_time, refund_item) <= 72 THEN 'Yes'
        ELSE 'No'
    END;
```
> we can also use ENUM ('Yes','No')as a datatype

7. Create a rank by buyer_id column in the transaction items table and filter for only the second
purchase per buyer.

```sql
SELECT
    *
FROM
    (SELECT
        *,
        RANK() OVER (PARTITION BY buyer_id ORDER BY purchase_time ASC) AS purchase_rank --rank function is used to give ranking on the basis of some conditions provided
    FROM
        transactions) t
WHERE
    t.purchase_rank = 2;
```

8. How will you find the second transaction time per buyer (don’t use min/max; assume there
were more transactions per buyer in the table)

```sql
SELECT
    t.buyer_id,
    t.purchase_time AS second_transaction_time
FROM
    (SELECT
        buyer_id,
        purchase_time,
        RANK() OVER (PARTITION BY buyer_id ORDER BY purchase_time ASC) AS purchase_rank
    FROM
        transactions) t
WHERE
    t.purchase_rank = 2;
--logic is almost similar as 7th 
```


