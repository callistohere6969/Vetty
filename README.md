# Vetty Assingment Questions

2. How many stores receive at least 5 orders/transactions in October 2020?
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

![Query Result]([https://raw.githubusercontent.com/your-username/your-repo/main/query_result.png](https://github.com/callistohere6969/Vetty/commit/d5e039e09d0be38d2f6d5aa96bff57a209cbaf84))

