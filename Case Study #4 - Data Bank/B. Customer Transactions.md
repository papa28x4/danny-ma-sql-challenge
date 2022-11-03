# 💵 Case Study #4 - Data Bank

## 🏦 Solution - B. Customer Transactions

**Schema (PostgreSQL v13)**

SET search_path = pizza_runner;

---
**1. What is the unique count and total amount for each transaction type?**
````sql
  SELECT txn_type, COUNT(txn_type), sum(txn_amount) AS total_amount
  FROM customer_transactions
  GROUP BY txn_type;
````
| txn_type   | count | total_amount |
| ---------- | ----- | ------------ |
| purchase   | 1617  | 806537       |
| deposit    | 2671  | 1359168      |
| withdrawal | 1580  | 793003       |

---
**2. What is the average total historical deposit counts and amounts for all customers?**
````sql
    WITH cte AS (
      SELECT AVG(txn_amount) AS cust_avg_amt, 
        COUNT(txn_amount) AS cust_deposit_count  
      FROM customer_transactions
      WHERE txn_type = 'deposit'
      GROUP BY customer_id
    )
    SELECT ROUND(AVG(cust_avg_amt), 2) AS avg_total, 
      ROUND(AVG(cust_deposit_count)) AS avg_deposit
    FROM cte;
````
| avg_total | avg_deposit |
| --------- | ----------- |
| 508.61    | 5           |

---
**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
````sql
  WITH cte AS (
    SELECT customer_id, Extract(MONTH FROM txn_date) AS month_number,
      TO_CHAR(txn_date, 'month' ) AS month_name,
    count(CASE WHEN txn_type = 'deposit' THEN 1 END) AS deposit,
     count(CASE WHEN txn_type = 'purchase' THEN 1 END) AS purchase,
    count(CASE WHEN txn_type = 'withdrawal' THEN 1 END) AS withdrawal
    FROM customer_transactions
    GROUP BY customer_id, month_number, month_name
    ORDER BY customer_id
  )
  SELECT month_name, Count(*) FROM cte
  WHERE deposit > 1 AND (purchase > 1 OR withdrawal > 1)
  GROUP BY month_number, month_name
  ORDER BY month_number;
````

| month_name | count |
| ---------- | ----- |
| january    | 88    |
| february   | 115   |
| march      | 137   |
| april      | 40    |

---
**4. What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.**
````sql
    WITH cte AS (
    		SELECT customer_id, EXTRACT(MONTH FROM txn_date) AS mth,
    		(DATE_TRUNC('month', txn_date) + INTERVAL '1 MONTH - 1 DAY') AS closing_date, txn_type, 
    			txn_amount
    		FROM customer_transactions
    		GROUP BY customer_id, mth, txn_type, txn_amount, txn_date
    		ORDER BY customer_id, mth
    	),
    	buffer AS (
    		SELECT DISTINCT customer_id,
    			('2020-01-31'::DATE + GENERATE_SERIES(0,3) * INTERVAL '1 MONTH') AS ending_month
    		FROM customer_transactions
    		ORDER BY customer_id, ending_month
    	),
    	buffer2 AS (
    		SELECT  b.customer_id, b.ending_month, EXTRACT(MONTH FROM ending_month) as month_number,
    			txn_type, txn_amount
    		FROM buffer b
    	 	LEFT JOIN cte c
    		ON b.customer_id = c.customer_id
    			 AND b.ending_month = c.closing_date
     		ORDER BY b.customer_id, ending_month
    	),
    	buffer3 AS (
    		SELECT customer_id, ending_month, month_number,
    			txn_type, txn_amount, 
    			(
    				CASE
    					WHEN txn_amount is null THEN 0
    					ELSE txn_amount
    				END
    			) AS cleaned_txn_amount
    		FROM buffer2
    	),
    	buffer4 AS (
    		SELECT *,
    			(CASE
    				WHEN txn_type = 'deposit' THEN cleaned_txn_amount
    				WHEN txn_type = 'purchase' THEN -cleaned_txn_amount
    				WHEN txn_type = 'withdrawal' THEN -cleaned_txn_amount
    				ELSE 0
    			END) AS signed_txn_amount FROM
    		buffer3
    	),
    	buffer5 AS(
    		SELECT customer_id, ending_month, month_number, SUM(signed_txn_amount) AS change_in_balance 
    		FROM buffer4
    		GROUP BY customer_id, month_number, ending_month
    	)
    	SELECT *, 
    	SUM(change_in_balance) 
    	OVER (PARTITION BY customer_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS monthly_balance
    	FROM buffer5
    	LIMIT 20;
````
| customer_id | ending_month             | month_number | change_in_balance | monthly_balance |
| ----------- | ------------------------ | ------------ | ----------------- | --------------- |
| 1           | 2020-01-31T00:00:00.000Z | 1            | 312               | 312             |
| 1           | 2020-02-29T00:00:00.000Z | 2            | 0                 | 312             |
| 1           | 2020-03-31T00:00:00.000Z | 3            | -952              | -640            |
| 1           | 2020-04-30T00:00:00.000Z | 4            | 0                 | -640            |
| 2           | 2020-01-31T00:00:00.000Z | 1            | 549               | 549             |
| 2           | 2020-02-29T00:00:00.000Z | 2            | 0                 | 549             |
| 2           | 2020-03-31T00:00:00.000Z | 3            | 61                | 610             |
| 2           | 2020-04-30T00:00:00.000Z | 4            | 0                 | 610             |
| 3           | 2020-01-31T00:00:00.000Z | 1            | 144               | 144             |
| 3           | 2020-02-29T00:00:00.000Z | 2            | -965              | -821            |
| 3           | 2020-03-31T00:00:00.000Z | 3            | -401              | -1222           |
| 3           | 2020-04-30T00:00:00.000Z | 4            | 493               | -729            |
| 4           | 2020-01-31T00:00:00.000Z | 1            | 848               | 848             |
| 4           | 2020-02-29T00:00:00.000Z | 2            | 0                 | 848             |
| 4           | 2020-03-31T00:00:00.000Z | 3            | -193              | 655             |
| 4           | 2020-04-30T00:00:00.000Z | 4            | 0                 | 655             |
| 5           | 2020-01-31T00:00:00.000Z | 1            | 954               | 954             |
| 5           | 2020-02-29T00:00:00.000Z | 2            | 0                 | 954             |
| 5           | 2020-03-31T00:00:00.000Z | 3            | -2877             | -1923           |
| 5           | 2020-04-30T00:00:00.000Z | 4            | -490              | -2413           |
