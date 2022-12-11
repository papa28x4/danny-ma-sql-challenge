# 🐟 Case Study #6 - Clique Bait

## 👩🏻‍💻 Solution - C. Product Funnel Analysis

**Schema (PostgreSQL v13)**

SET search_path = clique_bait;

### Using a single SQL query - create a new output table which has the following details: ###

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

````sql
    DROP TABLE IF EXISTS product_stats;
    CREATE TABLE product_stats AS
    WITH purchased_items AS (
      SELECT e1.* FROM events e1
      JOIN events e2
      ON e1.visit_id = e2.visit_id
        AND e1.event_type + 1 = e2.event_type
      WHERE e1.event_type = 2
      ORDER BY e1.visit_id
    ),
    purchase_summary AS (
      SELECT page_name AS product, COUNT(product_id) AS purchases FROM purchased_items
      JOIN page_hierarchy
      USING (page_id)
      GROUP BY product_id, page_name
    ),
    views_adds AS (
      SELECT page_name AS product, product_category, COUNT(CASE WHEN event_type = 1 THEN 1 END) AS page_views, 
        COUNT(CASE WHEN event_type = 2 THEN 1 END) AS cart_adds
      FROM events
      JOIN page_hierarchy
      USING (page_id)
      WHERE event_type IN (1, 2) 
        AND product_id IS NOT NULL
      GROUP BY page_name, product_category
    )
    SELECT *, (cart_adds - purchases) AS abandoned FROM views_adds
    JOIN purchase_summary
    USING (product)
    ORDER BY product_category;
````
````sql
    SELECT * FROM product_stats;
````
| product        | product_category | page_views | cart_adds | purchases | abandoned |
| -------------- | ---------------- | ---------- | --------- | --------- | --------- |
| Tuna           | Fish             | 1515       | 931       | 697       | 234       |
| Kingfish       | Fish             | 1559       | 920       | 707       | 213       |
| Salmon         | Fish             | 1559       | 938       | 711       | 227       |
| Black Truffle  | Luxury           | 1469       | 924       | 707       | 217       |
| Russian Caviar | Luxury           | 1563       | 946       | 697       | 249       |
| Lobster        | Shellfish        | 1547       | 968       | 754       | 214       |
| Abalone        | Shellfish        | 1525       | 932       | 699       | 233       |
| Oyster         | Shellfish        | 1568       | 943       | 726       | 217       |
| Crab           | Shellfish        | 1564       | 949       | 719       | 230       |



---
**Additionally, create another table which further aggregates the data for the above points but this 
	time for each product category instead of individual products.**
````sql
    DROP TABLE IF EXISTS product_category_agg;
    CREATE TABLE product_category_agg AS
    SELECT product_category, SUM(page_views) AS page_views, SUM(cart_adds) AS cart_adds,
      SUM(purchases) AS purchases, SUM(abandoned) AS abandoned
    FROM product_stats
    GROUP BY product_category;
````
````sql
    SELECT * FROM product_category_agg;
````
| product_category | page_views | cart_adds | purchases | abandoned |
| ---------------- | ---------- | --------- | --------- | --------- |
| Luxury           | 3032       | 1870      | 1404      | 466       |
| Shellfish        | 6204       | 3792      | 2898      | 894       |
| Fish             | 4633       | 2789      | 2115      | 674       |

---
### Use your 2 new output tables - answer the following questions: ##

***1. Which product had the most views, cart adds and purchases?***

***2. Which product was most likely to be abandoned?***

    WITH cte AS (
      SELECT product, page_views, RANK() OVER(ORDER BY page_views DESC) AS page_views_ranking,
        cart_adds, RANK() OVER(ORDER BY cart_adds DESC) AS cart_adds_ranking,
        purchases, RANK() OVER(ORDER BY purchases DESC) AS purchases_ranking,
        abandoned, RANK() OVER(ORDER BY abandoned DESC) AS abandoned_ranking
      FROM product_stats
      ORDER BY page_views DESC
    )
    SELECT * FROM cte
    WHERE page_views_ranking = 1 OR cart_adds_ranking = 1 
      OR purchases_ranking = 1 OR abandoned_ranking = 1;

| product        | page_views | page_views_ranking | cart_adds | cart_adds_ranking | purchases | purchases_ranking | abandoned | abandoned_ranking |
| -------------- | ---------- | ------------------ | --------- | ----------------- | --------- | ----------------- | --------- | ----------------- |
| Oyster         | 1568       | 1                  | 943       | 4                 | 726       | 2                 | 217       | 6                 |
| Russian Caviar | 1563       | 3                  | 946       | 3                 | 697       | 8                 | 249       | 1                 |
| Lobster        | 1547       | 6                  | 968       | 1                 | 754       | 1                 | 214       | 8                 |

---
***3. Which product had the highest view to purchase percentage?***
````sql
    SELECT product, page_views, purchases, 
    	ROUND(purchases::numeric/page_views * 100, 2) AS conversion
    FROM product_stats
    ORDER BY conversion DESC
    LIMIT 1;
````
| product | page_views | purchases | conversion |
| ------- | ---------- | --------- | ---------- |
| Lobster | 1547       | 754       | 48.74      |

---
**4. What is the average conversion rate from view to cart add?**
````sql
    SELECT ROUND(AVG(cart_adds::numeric/page_views * 100), 2) AS view_to_cart_add_conversion
    FROM product_stats;
````
| view_to_cart_add_conversion |
| --------------------------- |
| 60.95                       |

---
**5. What is the average conversion rate from cart add to purchase?**
````sql
    SELECT ROUND(AVG(purchases::numeric/cart_adds * 100), 2) AS cart_add_to_purchase_conversion
    FROM product_stats;
````
| cart_add_to_purchase_conversion |
| ------------------------------- |
| 75.93                           |
