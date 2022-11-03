# 💵 Case Study #4 - Data Bank

## 🏦 Solution - A. Customer Nodes Exploration

**Schema (PostgreSQL v13)**

SET search_path = pizza_runner;



---
**1. How many unique nodes are there on the Data Bank system?**
````sql
    SELECT COUNT(DISTINCT node_id) FROM customer_nodes;
````
| count |
| ----- |
| 5     |

---
**2. What is the number of nodes per region?**
````sql
    SELECT region_id, region_name, COUNT(DISTINCT node_id) 
    FROM customer_nodes
    JOIN regions
    USING (region_id)
    GROUP BY region_id, region_name
    ORDER BY region_id;
````
| region_id | region_name | count |
| --------- | ----------- | ----- |
| 1         | Australia   | 5     |
| 2         | America     | 5     |
| 3         | Africa      | 5     |
| 4         | Asia        | 5     |
| 5         | Europe      | 5     |

---
**3. How many customers are allocated to each region?**
````sql
    SELECT region_id, region_name, COUNT(DISTINCT customer_id) 
    FROM customer_nodes
    JOIN regions
    USING (region_id)
    GROUP BY region_id, region_name
    ORDER BY region_id;
````
| region_id | region_name | count |
| --------- | ----------- | ----- |
| 1         | Australia   | 110   |
| 2         | America     | 105   |
| 3         | Africa      | 102   |
| 4         | Asia        | 95    |
| 5         | Europe      | 88    |

---
**4. How many days on average are customers reallocated to a different node?**
````sql
    WITH ranking AS (
        SELECT customer_id, start_date, end_date, 
        RANK() OVER(PARTITION BY customer_id ORDER BY end_date) 
        FROM customer_nodes
    )
    SELECT avg(r2.start_date - r1.start_date) FROM ranking r1
    JOIN ranking r2
    ON r1.customer_id = r2.customer_id
        AND r2.rank - 1 = r1.rank;
````
| avg                 |
| ------------------- |
| 15.6340000000000000 |




---
**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**
````sql
  CREATE VIEW reallocations AS
    WITH ranking AS (
        SELECT customer_id, region_name, start_date, end_date, 
        RANK() OVER(PARTITION BY customer_id ORDER BY end_date) 
        FROM customer_nodes
        JOIN regions
        USING (region_id)
    )
    SELECT r1.customer_id, r1.region_name, (r2.start_date - r1.start_date) AS diff FROM ranking r1
    JOIN ranking r2
    ON r1.customer_id = r2.customer_id
        AND r2.rank - 1 = r1.rank;
   ````
   ````sql
    SELECT region_name,
        PERCENTILE_CONT(0.5) WITHIN
            GROUP (ORDER BY
              diff) AS median,
        PERCENTILE_CONT(0.8) WITHIN
            GROUP (ORDER BY
              diff) AS _80th,
        PERCENTILE_CONT(0.95) WITHIN
            GROUP (ORDER BY
              diff) AS _95th
    FROM reallocations
    GROUP BY region_name;
````
| region_name | median | _80th | _95th |
| ----------- | ------ | ----- | ----- |
| Africa      | 16     | 25    | 29    |
| America     | 16     | 24    | 29    |
| Asia        | 16     | 24    | 29    |
| Australia   | 16     | 24    | 29    |
| Europe      | 16     | 25    | 29    |
