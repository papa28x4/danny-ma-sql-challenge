# 🍜 Case Study #1: Danny's Diner

## Solutions

**Schema (PostgreSQL v13)**
     
     SET search_path = pizza_runner;
---
### 1. What is the total amount each customer spent at the restaurant?

````sql
     SELECT customer_id, SUM(price) AS total_amount
     FROM sales
     JOIN menu
     USING (product_id)
     GROUP BY customer_id;
````
| customer_id | total_amount |
| ----------- | ------------ |
| B           | 74           |
| C           | 36           |
| A           | 76           |

---
### 2. How many days has each customer visited the restaurant?
````sql
    SELECT customer_id, COUNT(DISTINCT(order_date)) AS visits
    FROM sales
    GROUP BY customer_id;
````
| customer_id | visits |
| ----------- | ------ |
| A           | 4      |
| B           | 6      |
| C           | 2      |

---
### 3. What was the first item from the menu purchased by each customer?
````sql
    WITH ranking as (
        SELECT *, 
          DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) as RANK
        FROM sales
        JOIN menu
        USING (product_id)
    )
     SELECT customer_id, product_name, order_date
     FROM ranking 
     WHERE rank = 1
     GROUP BY customer_id, product_name, order_date;
````
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| B           | curry        | 2021-01-01T00:00:00.000Z |
| C           | ramen        | 2021-01-01T00:00:00.000Z |

---
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
    SELECT product_name, count(product_id) AS product_count
    FROM sales
    JOIN menu
    USING (product_id)
    GROUP BY product_name
    ORDER BY product_count DESC
    LIMIT 1;
````
| product_name | product_count |
| ------------ | ------------- |
| ramen        | 8             |

---
### 5. Which item was the most popular for each customer?
````sql
    WITH ranking AS
    (
      SELECT customer_id, product_name, COUNT(product_id) AS order_count,
          DENSE_RANK() Over(PARTITION BY customer_id ORDER BY COUNT(product_id) desc)
       FROM menu 
       JOIN sales 
       USING (product_id)
       GROUP BY customer_id, product_name
    )
    SELECT customer_id, product_name, order_count
    FROM ranking where dense_rank = 1;
````
| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

---
### 6. Which item was purchased first by the customer after they became a member?
````sql
    with ranking as (
        SELECT *, 
            DENSE_RANK() OVER(partition by customer_id order by order_date)
        FROM sales s
        JOIN members m
        USING (customer_id)
        WHERE s.order_date >= m.join_date
    )
    SELECT customer_id, order_date, product_name from ranking
    JOIN menu 
    USING (product_id)
    WHERE dense_rank = 1;
````
| customer_id | order_date               | product_name |
| ----------- | ------------------------ | ------------ |
| B           | 2021-01-11T00:00:00.000Z | sushi        |
| A           | 2021-01-07T00:00:00.000Z | curry        |

---
### 7. Which item was purchased just before the customer became a member?
````sql
    with ranking as (
        SELECT *,
            DENSE_RANK() OVER(partition by customer_id order by order_date desc)
        FROM sales s
        JOIN members m
        USING (customer_id)
        WHERE s.order_date < m.join_date
    )
    SELECT customer_id, order_date, product_name 
    FROM ranking 
    JOIN menu
    USING (product_id)
    WHERE dense_rank = 1;
````
| customer_id | order_date               | product_name |
| ----------- | ------------------------ | ------------ |
| B           | 2021-01-04T00:00:00.000Z | sushi        |
| A           | 2021-01-01T00:00:00.000Z | sushi        |
| A           | 2021-01-01T00:00:00.000Z | curry        |

---
### 8. What is the total items and amount spent for each member before they became a member?
````sql
    SELECT customer_id, COUNT(DISTINCT(product_id)) as total_items, 
     SUM(price) as amount_spent	
    FROM sales s
    JOIN members m
    USING (customer_id)
    JOIN menu
    USING (product_id)
    WHERE s.order_date < m.join_date
    GROUP BY customer_id;
````
| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 2           | 40           |

---
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
````sql
    WITH customer_points AS (
        SELECT *, 
            (
                CASE
                    WHEN product_name = 'sushi' THEN price * 10 * 2
                    ELSE price * 10	
                END
            ) AS points
        FROM sales
        JOIN menu
        USING (product_id)
    )
    SELECT customer_id, SUM(points) FROM customer_points
    GROUP BY customer_id
    ORDER BY customer_id;
````
| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

---
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
````sql
    WITH points_calc AS (
         SELECT *,
            (CASE
                WHEN  s.order_date - m.join_date >= 0 and s.order_date - m.join_date <= 6 THEN price * 10 * 2
                WHEN product_name = 'sushi' THEN price * 10 * 2
                ELSE  price * 10
            END) as points
          FROM sales s
          JOIN menu mu
          USING (product_id)
          JOIN members m
          USING (customer_id)
          WHERE EXTRACT(MONTH FROM order_date) = 1 AND EXTRACT(YEAR FROM order_date) = 2021
    )
    SELECT customer_id, SUM(points)
    FROM points_calc
    GROUP BY customer_id;
````
| customer_id | sum  |
| ----------- | ---- |
| A           | 1370 |
| B           | 820  |

---
