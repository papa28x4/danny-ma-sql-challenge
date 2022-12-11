# 👗 Case Study #7 - Balanced Tree Clothing Co.

## 🍝 Solution - C. Product Analysis

**Schema (PostgreSQL v13)**

    SET search_path = balanced_tree;

---
**1. What are the top 3 products by total revenue before discount?**
````sql
    SELECT prod_id, product_name, SUM(qty * s.price) AS revenue 
    FROM sales s
    JOIN product_details p
    ON s.prod_id = p.product_id
    GROUP BY prod_id, product_name
    ORDER BY revenue DESC
    LIMIT 3;
````
| prod_id | product_name                 | revenue |
| ------- | ---------------------------- | ------- |
| 2a2353  | Blue Polo Shirt - Mens       | 217683  |
| 9ec847  | Grey Fashion Jacket - Womens | 209304  |
| 5d267b  | White Tee Shirt - Mens       | 152000  |

---
**2. What is the total quantity, revenue and discount for each segment?**
````sql
    SELECT segment_name, SUM(qty) AS total_qty, 
     SUM(qty * s.price) AS total_revenue, 
     ROUND(SUM(qty * s.price * discount::NUMERIC/100), 2) AS total_discount
    FROM sales s
    JOIN product_details p
    ON s.prod_id = p.product_id
    GROUP BY segment_name
    ORDER BY segment_name;
````
| segment_name | total_qty | total_revenue | total_discount |
| ------------ | --------- | ------------- | -------------- |
| Jacket       | 11385     | 366983        | 44277.46       |
| Jeans        | 11349     | 208350        | 25343.97       |
| Shirt        | 11265     | 406143        | 49594.27       |
| Socks        | 11217     | 307977        | 37013.44       |

---
**3. What is the top selling product for each segment?**
````sql
    WITH cte AS (
     SELECT segment_name, product_name, 
     SUM(qty) AS total_qty,
     RANK() OVER(PARTITION BY segment_name ORDER BY SUM(qty) DESC)
     FROM sales s
     JOIN product_details p
     ON s.prod_id = p.product_id
     GROUP BY segment_name, product_name
    )
    SELECT segment_name, product_name, total_qty 
    FROM cte
    WHERE rank = 1;
````
| segment_name | product_name                  | total_qty |
| ------------ | ----------------------------- | --------- |
| Jacket       | Grey Fashion Jacket - Womens  | 3876      |
| Jeans        | Navy Oversized Jeans - Womens | 3856      |
| Shirt        | Blue Polo Shirt - Mens        | 3819      |
| Socks        | Navy Solid Socks - Mens       | 3792      |

---
**4. What is the total quantity, revenue and discount for each category?**
````sql
    SELECT category_name, SUM(qty) AS total_qty, 
     SUM(qty * s.price) AS total_revenue, 
     ROUND(SUM(qty * s.price * discount::NUMERIC/100), 2) AS total_discount
    FROM sales s
    JOIN product_details p
    ON s.prod_id = p.product_id
    GROUP BY category_name
    ORDER BY category_name;
````
| category_name | total_qty | total_revenue | total_discount |
| ------------- | --------- | ------------- | -------------- |
| Mens          | 22482     | 714120        | 86607.71       |
| Womens        | 22734     | 575333        | 69621.43       |

---
**5. What is the top selling product for each category?**
````sql
    WITH cte AS (
     SELECT category_name, product_name, 
     SUM(qty) AS total_qty,
     RANK() OVER(PARTITION BY category_name ORDER BY SUM(qty) DESC)
     FROM sales s
     JOIN product_details p
     ON s.prod_id = p.product_id
     GROUP BY category_name, product_name
    )
    SELECT category_name, product_name, total_qty 
    FROM cte
    WHERE rank = 1;
````
| category_name | product_name                 | total_qty |
| ------------- | ---------------------------- | --------- |
| Mens          | Blue Polo Shirt - Mens       | 3819      |
| Womens        | Grey Fashion Jacket - Womens | 3876      |

---
**6. What is the percentage split of revenue by product for each segment?**
````sql
    WITH cte AS (
     SELECT segment_name, product_name, 
     SUM(qty * s.price) AS product_revenue
     FROM sales s
     JOIN product_details p
     ON s.prod_id = p.product_id
     GROUP BY segment_name, product_name
    )
    SELECT segment_name, product_name, 
     ROUND(100 * product_revenue::NUMERIC/SUM(product_revenue) OVER(PARTITION BY segment_name), 2) 
     AS percentage_split
    FROM cte;
````
| segment_name | product_name                     | percentage_split |
| ------------ | -------------------------------- | ---------------- |
| Jacket       | Indigo Rain Jacket - Womens      | 19.45            |
| Jacket       | Khaki Suit Jacket - Womens       | 23.51            |
| Jacket       | Grey Fashion Jacket - Womens     | 57.03            |
| Jeans        | Navy Oversized Jeans - Womens    | 24.06            |
| Jeans        | Black Straight Jeans - Womens    | 58.15            |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.79            |
| Shirt        | White Tee Shirt - Mens           | 37.43            |
| Shirt        | Blue Polo Shirt - Mens           | 53.60            |
| Shirt        | Teal Button Up Shirt - Mens      | 8.98             |
| Socks        | Navy Solid Socks - Mens          | 44.33            |
| Socks        | White Striped Socks - Mens       | 20.18            |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.50            |

---
**7. What is the percentage split of revenue by segment for each category?**
````sql
    WITH cte AS (
     SELECT category_name, segment_name, 
     SUM(qty * s.price) AS segment_revenue
     FROM sales s
     JOIN product_details p
     ON s.prod_id = p.product_id
     GROUP BY category_name, segment_name
    )
    SELECT category_name, segment_name, 
     ROUND(100 * segment_revenue::NUMERIC/SUM(segment_revenue) OVER(PARTITION BY category_name), 2) 
     AS percentage_split
    FROM cte;
````
| category_name | segment_name | percentage_split |
| ------------- | ------------ | ---------------- |
| Mens          | Socks        | 43.13            |
| Mens          | Shirt        | 56.87            |
| Womens        | Jeans        | 36.21            |
| Womens        | Jacket       | 63.79            |

---
**8. What is the percentage split of total revenue by category?**
````sql
    SELECT category_name, 
     ROUND(100 * SUM(qty * s.price)::NUMERIC / (SELECT SUM(qty * price) FROM sales), 2) 
     AS percentage_split
    FROM sales s
    JOIN product_details p
    ON s.prod_id = p.product_id
    GROUP BY category_name;
````
| category_name | percentage_split |
| ------------- | ---------------- |
| Mens          | 55.38            |
| Womens        | 44.62            |

---
**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**
````sql
    SELECT product_name, 
     ROUND(100 * COUNT(prod_id)::NUMERIC / (SELECT COUNT(DISTINCT txn_id) FROM sales), 2)
    FROM sales s
    JOIN product_details p
    ON s.prod_id = p.product_id 
    GROUP BY product_name;
````
| product_name                     | round |
| -------------------------------- | ----- |
| White Tee Shirt - Mens           | 50.72 |
| Navy Solid Socks - Mens          | 51.24 |
| Grey Fashion Jacket - Womens     | 51.00 |
| Navy Oversized Jeans - Womens    | 50.96 |
| Pink Fluro Polkadot Socks - Mens | 50.32 |
| Khaki Suit Jacket - Womens       | 49.88 |
| Black Straight Jeans - Womens    | 49.84 |
| White Striped Socks - Mens       | 49.72 |
| Blue Polo Shirt - Mens           | 50.72 |
| Indigo Rain Jacket - Womens      | 50.00 |
| Cream Relaxed Jeans - Womens     | 49.72 |
| Teal Button Up Shirt - Mens      | 49.68 |

---
**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**
