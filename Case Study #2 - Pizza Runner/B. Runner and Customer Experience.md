
# 🍕 Case Study #2 Pizza Runner

## Solution - B. Runner and Customer Experience

**Schema (PostgreSQL v13)**

    SET search_path = pizza_runner;
    
   
---
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
    SELECT DATE_PART('week', registration_date + 3) AS week_of_year,
	COUNT(runner_id)
    FROM runners
    GROUP BY DATE_PART('week', registration_date + 3)
    ORDER BY DATE_PART('week', registration_date + 3);
````

| week_of_year | reg_count |
| ------------ | --------- |
| 1            | 2         |
| 2            | 1         |
| 3            | 1         |

---
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

**Approach 1: Find the average first and then round up to nearest minute**

````sql
   WITH order_time_diff AS (
      SELECT DISTINCT order_id, (pickup_time - order_time) AS time_diff 
      FROM delivered_orders
   )
   SELECT date_part('minute', AVG(time_diff) + '30 second' ) AS avg_mins
   FROM order_time_diff;
````

| avg_mins   |
| ---------- |
| 16         |

**Approach 2: Find the difference in minutes and then calculate the average**

````sql
 WITH each_runner AS
       (
    	SELECT c.order_id, c.order_time,r.pickup_time,
    	DATE_PART('minute', r.pickup_time - c.order_time) AS diff_in_minutes
    	FROM customer_orders_temp AS c
    	JOIN runner_orders_temp as r
        ON r.order_id = c.order_id
    	WHERE r.distance <> 0
    	GROUP BY  c.order_id, order_time,r.pickup_time
       )
      SELECT ROUND(AVG(diff_in_minutes)) AS avg_mins
      FROM each_runner;
````
| avg_mins |
| -------- |
| 16       |

---
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
   WITH orders_group AS (
      SELECT order_id, count(order_id) AS order_count, 
         (pickup_time - order_time) AS time_diff 
      FROM delivered_orders
      GROUP BY order_id, pickup_time, order_time
      ORDER BY order_id
   )
   SELECT order_count, AVG(time_diff) 
   FROM orders_group
   GROUP BY order_count;
````

| order_count | avg        |
| ----------- | ---------- |
| 3           | 00:29:17   |
| 2           | 00:18:22.5 |
| 1           | 00:12:21.4 |

---
### 4. What was the average distance travelled for each customer?

````sql
    SELECT customer_id, ROUND(AVG(distance), 2) AS avg_distance
    FROM delivered_orders
    GROUP BY customer_id
    ORDER BY customer_id;
````

| customer_id | avg_distance |
| ----------- | ------------ |
| 101         | 20.00        |
| 102         | 16.73        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

---
### 5. What was the difference between the longest and shortest delivery times for all orders?

````sql
    SELECT MAX(duration) - MIN(duration) AS delivery_time_diff FROM runner_orders_temp;
````

| delivery_time_diff |
| ------------------ |
| 30                 |


---
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
   SELECT DISTINCT order_id, runner_id,  
      round(distance / (duration::numeric/60), 2) AS average_speed
   FROM delivered_orders
   ORDER BY runner_id, average_speed;
````

| order_id | runner_id | average_speed |
| -------- | --------- | ------------- |
| 1        | 1         | 37.50         |
| 3        | 1         | 40.20         |
| 2        | 1         | 44.44         |
| 10       | 1         | 60.00         |
| 4        | 2         | 35.10         |
| 7        | 2         | 60.00         |
| 8        | 2         | 93.60         |
| 5        | 3         | 40.00         |

---
### 7. What is the successful delivery percentage for each runner?

````sql
   SELECT runner_id,  round(count(distance)::numeric/ count(runner_id) * 100) AS delivery_percentage
   FROM runner_orders_temp
   GROUP BY runner_id;
````

| runner_id | delivery_percentage |
| --------- | ------------------- |
| 3         | 50                  |
| 2         | 75                  |
| 1         | 100                 |

---


