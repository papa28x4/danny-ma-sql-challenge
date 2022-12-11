# 🐟 Case Study #6 - Clique Bait

## 👩🏻‍💻 Solution - B. Digital Analysis

**Schema (PostgreSQL v13)**

    SET search_path = clique_bait;



---
**1. How many users are there?**
````sql
    SELECT COUNT(DISTINCT user_id) FROM users;
````
| count |
| ----- |
| 500   |

---
**2. How many cookies does each user have on average?**
````sql
    SELECT ROUND(COUNT(cookie_id)::NUMERIC/COUNT(DISTINCT user_id)) 
    FROM users;
````
| round |
| ----- |
| 4     |

---
**3. What is the unique number of visits by all users per month?**
````sql
    SELECT COUNT(DISTINCT visit_id), EXTRACT(MONTH FROM event_time) AS month_number, 
        TO_CHAR(event_time, 'month') AS month_name
    FROM events
    GROUP BY month_number, month_name
    ORDER BY month_number;
````
| count | month_number | month_name |
| ----- | ------------ | ---------- |
| 876   | 1            | january    |
| 1488  | 2            | february   |
| 916   | 3            | march      |
| 248   | 4            | april      |
| 36    | 5            | may        |

---
**4. What is the number of events for each event type?**
````sql
    SELECT event_type, event_name, COUNT(event_type) FROM events
    JOIN event_identifier
    USING (event_type)
    GROUP BY event_type, event_name
    ORDER BY event_type;
````
| event_type | event_name    | count |
| ---------- | ------------- | ----- |
| 1          | Page View     | 20928 |
| 2          | Add to Cart   | 8451  |
| 3          | Purchase      | 1777  |
| 4          | Ad Impression | 876   |
| 5          | Ad Click      | 702   |

---
**5. What is the percentage of visits which have a purchase event?**
````sql
   SELECT ROUND((100 * COUNT(CASE WHEN event_name='Purchase' THEN 1 END)::NUMERIC
                 /COUNT(DISTINCT visit_id))) AS purchase_percentage
   FROM events
   JOIN event_identifier
   USING (event_type);
````
| purchase_percentage |
| ------------------- |
| 50                  |

---
**6. What is the percentage of visits which view the checkout page but do not have a purchase event?**

***Approach A***
````sql
   WITH visits AS (
       SELECT DISTINCT visit_id FROM events
       JOIN page_hierarchy
       USING (page_id)
       WHERE page_name = 'Checkout'
  ),
  purchases AS (
    SELECT count(1) AS buy_count FROM events
    JOIN visits
    USING (visit_id)
    WHERE event_type = 3
  )
  SELECT ROUND(100 * ((SELECT COUNT(visit_id) FROM visits) - (SELECT buy_count FROM purchases))::NUMERIC
            / (SELECT COUNT(1) FROM visits), 2);
````
| round |
| ----- |
| 15.50 |

---
***Approach B***
````sql
    WITH checkout_purchase AS (
        SELECT 
          visit_id,
          MAX(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
          MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
        FROM clique_bait.events
        GROUP BY visit_id
    )
    SELECT 
      ROUND(100 * (1-(SUM(purchase)::numeric/SUM(checkout))),2) AS percentage_checkout_view_with_no_purchase
    FROM checkout_purchase;
````
| percentage_checkout_view_with_no_purchase |
| ----------------------------------------- |
| 15.50                                     |

---
**7. What are the top 3 pages by number of views?**
````sql
    SELECT page_name, COUNT(page_id) AS view_count
    FROM events
    JOIN page_hierarchy
    USING (page_id)
    WHERE event_type = 1 
    GROUP BY page_name
    ORDER BY view_count desc
    LIMIT 3;
````
| page_name    | view_count |
| ------------ | ---------- |
| All Products | 3174       |
| Checkout     | 2103       |
| Home Page    | 1782       |

---
**8. What is the number of views and cart adds for each product category?**
````sql
    SELECT product_category, COUNT(CASE WHEN event_type = 1 THEN 1 END) AS view_count,
        COUNT(CASE WHEN event_type = 2 THEN 1 END) AS cart_adds
    FROM page_hierarchy
    JOIN events
    USING (page_id)
    GROUP BY product_category
    HAVING product_category IS NOT NULL
    ORDER BY view_count DESC;
````
| product_category | view_count | cart_adds |
| ---------------- | ---------- | --------- |
| Shellfish        | 6204       | 3792      |
| Fish             | 4633       | 2789      |
| Luxury           | 3032       | 1870      |

---
**9. What are the top 3 products by purchases?**
````sql
    WITH cte AS (
        SELECT e1.* FROM events e1
        JOIN events e2
        ON e1.visit_id = e2.visit_id
            AND e1.event_type + 1 = e2.event_type
        WHERE e1.event_type = 2
        ORDER BY e1.visit_id
    )
    SELECT page_name AS product, COUNT(product_id) AS purchases FROM cte
    JOIN page_hierarchy
    USING (page_id)
    GROUP BY product_id, page_name
    ORDER BY purchases DESC
    LIMIT 3;
````
| product | purchases |
| ------- | --------- |
| Lobster | 754       |
| Oyster  | 726       |
| Crab    | 719       |

---
