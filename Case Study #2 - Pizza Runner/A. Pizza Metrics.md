# 🍕 Case Study #2 - Pizza Runner

## 🍝 Solution - A. Pizza Metrics

**Schema (PostgreSQL v13)**

SET search_path = pizza_runner;
   
---
### 1. How many pizzas were ordered?

````sql
    SELECT COUNT(pizza_id) AS ordered_pizza FROM customer_orders_temp;
````
| ordered_pizza |
| ------------- |
| 14            |

---
### 2. How many unique customer orders were made?

````sql
    SELECT COUNT(DISTINCT(order_id)) AS unique_customer_orders 
    FROM customer_orders_temp;
````
| unique_customer_orders |
| ---------------------- |
| 10                     |

---
### 3. How many successful orders were delivered by each runner?

````sql
    SELECT runner_id, COUNT(runner_id) AS order_count 
    FROM runner_orders_temp
    WHERE DISTANCE IS NOT NULL
    GROUP BY runner_id;
````
| runner_id | order_count |
| --------- | ----------- |
| 3         | 1           |
| 2         | 3           |
| 1         | 4           |

---
### 4. How many of each type of pizza was delivered?

````sql
   SELECT pizza_name, COUNT(pizza_id) 
   FROM runner_orders_temp
   JOIN customer_orders_temp
   USING (order_id)
   JOIN pizza_names
   USING (pizza_id)
   WHERE DISTANCE IS NOT NULL
   GROUP BY pizza_name;
````
| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |

---
### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
   SELECT customer_id, pizza_name, COUNT(customer_id) AS pizza_count
   FROM customer_orders_temp
   JOIN pizza_names
   USING (pizza_id)
   GROUP BY customer_id, pizza_name
   ORDER BY customer_id;
````
| customer_id | pizza_name | pizza_count |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

---
### 6. What was the maximum number of pizzas delivered in a single order?

````sql
    With ranking AS (
      SELECT order_id, COUNT(order_id) AS pizza_count,
         RANK() OVER(ORDER BY COUNT(order_id) DESC)
      FROM customer_orders_temp
      JOIN runner_orders_temp
      USING (order_id)
      WHERE DISTANCE IS NOT NULL
      GROUP BY order_id
   )
   SELECT order_id, pizza_count FROM ranking
   WHERE rank = 1;
````
| order_id | pizza_count |
| -------- | ----------- |
| 4        | 3           |

---
**A good number of questions will often require us to join the customer_orders_temp and runner_orders_temp tables.
	Hence, we can create a view so we do not have keep writing same join queries**

````sql
    	CREATE VIEW delivered_orders AS
	SELECT * FROM customer_orders_temp
	JOIN runner_orders_temp	
	USING (order_id)
	WHERE distance IS NOT NULL;
````


---
### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
   SELECT customer_id,
	COUNT(
		CASE 
			WHEN exclusions <> '' OR extras <> ''  THEN 1
		END
	) AS changed,
	COUNT(
		CASE 
			WHEN exclusions = '' AND extras = ''  THEN 1
		END
	) AS unchanged	
   FROM delivered_orders
   GROUP BY customer_id
   ORDER BY customer_id;
````
| customer_id | changed | unchanged |
| ----------- | ------- | --------- |
| 101         | 0       | 2         |
| 102         | 0       | 3         |
| 103         | 3       | 0         |
| 104         | 2       | 1         |
| 105         | 1       | 0         |

---
### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
   SELECT COUNT(*) AS pizza_having_exclusions_n_extras
   FROM delivered_orders
   WHERE exclusions <> ''  
   AND extras <> '';
````
| pizza_having_exclusions_n_extras |
| -------------------------------- |
| 1                                |

---
### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
    SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day, 
    	COUNT(pizza_id) AS pizza_count
    FROM customer_orders
    GROUP BY hour_of_day
    ORDER BY hour_of_day;
````

| hour_of_day | pizza_count |
| ----------- | ----------- |
| 11          | 1           |
| 13          | 3           |
| 18          | 3           |
| 19          | 1           |
| 21          | 3           |
| 23          | 3           |

---
### 10. What was the volume of orders for each day of the week?

````sql
   SELECT TO_CHAR(order_time, 'day') AS day_of_week,
    	COUNT(pizza_id) AS pizza_count
   FROM customer_orders
   GROUP BY day_of_week
   ORDER BY day_of_week;
````
| day_of_week | pizza_count |
| ----------- | ----------- |
| friday      | 1           |
| saturday    | 5           |
| thursday    | 3           |
| wednesday   | 5           |

---
