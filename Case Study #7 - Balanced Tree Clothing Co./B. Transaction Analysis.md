# 👗 Case Study #7 - Balanced Tree Clothing Co.

## 🍝 Solution - B. Transaction Analysis

**Schema (PostgreSQL v13)**

    SET search_path = balanced_tree;
    
---
**1. How many unique transactions were there?**
````sql
    SELECT COUNT(DISTINCT txn_id) AS unique_txn FROM sales;
````
| unique_txn |
| ---------- |
| 2500       |

---
**2. What is the average unique products purchased in each transaction?**
````sql
    WITH cte AS (
        SELECT txn_id, COUNT(DISTINCT prod_id) AS unique_prod_count
        FROM sales
        GROUP BY txn_id
    )
    SELECT ROUND(AVG(unique_prod_count), 2) AS avg_unique_prod_count FROM cte;
````
| avg_unique_prod_count |
| --------------------- |
| 6.04                  |

---
**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**
````sql
    WITH cte AS (
        SELECT (qty * price) AS unit_revenue FROM sales
    )
    SELECT  
        PERCENTILE_CONT(0.25) WITHIN
            GROUP (ORDER BY
              unit_revenue) AS _25th,
        PERCENTILE_CONT(0.50) WITHIN
            GROUP (ORDER BY
              unit_revenue DESC) AS median,
        PERCENTILE_CONT(0.75) WITHIN
            GROUP (ORDER BY
              unit_revenue) AS _75th
    FROM cte;
````
| _25th | median | _75th |
| ----- | ------ | ----- |
| 38    | 65     | 116   |

---
**4. What is the average discount value per transaction?**
````sql
    SELECT ROUND(AVG(discount),2) AS avg_discount FROM sales;
````
| avg_discount |
| ------------ |
| 12.10        |

---
**5. What is the percentage split of all transactions for members vs non-members?**
````sql
    SELECT member, 
        ROUND(100 * COUNT(txn_id)::numeric / (SELECT COUNT(txn_id) from sales), 2) AS txn_percentage 
    FROM sales
    GROUP BY member;
````
| member | txn_percentage |
| ------ | -------------- |
| false  | 39.97          |
| true   | 60.03          |

---
**6. What is the average revenue for member transactions and non-member transactions?**
````sql
    SELECT member, 
        ROUND(AVG(qty * price * (1 - discount::numeric/100)), 2) AS  total_discounted_revenue 
    FROM sales
    GROUP BY member;
````
| member | total_discounted_revenue |
| ------ | ------------------------ |
| false  | 74.54                    |
| true   | 75.43                    |

---
