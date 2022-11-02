# 🍕 Case Study #2 Pizza Runner

## Solution - D. Pricing and Ratings


**Schema (PostgreSQL v13)**

   
    SET search_path = pizza_runner;
   
---
### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
````sql
    WITH calc_prices AS (
      SELECT pizza_id, pizza_name,
         (
            CASE
               WHEN pizza_name = 'Meatlovers' THEN 12
               WHEN pizza_name = 'Vegetarian' THEN 10
            END
         ) AS cost
      FROM delivered_orders
      JOIN pizza_names
      USING (pizza_id)
   )
   SELECT sum(cost) AS revenue FROM calc_prices;
 ````
| revenue |
| ------- |
| 138     |

---
### 2A. What if there was an additional $1 charge for any pizza extras?
````sql
    WITH calc_prices AS (
      SELECT order_id, customer_id, pizza_id, pizza_name, 
         (
               CASE
                  WHEN pizza_name = 'Meatlovers' THEN 12
                  WHEN pizza_name = 'Vegetarian' THEN 10
               END
         ) AS cost,
         array_length(string_to_array( extras, ', ' ), 1) AS extra_cost
      FROM delivered_orders
      JOIN pizza_names
      USING (pizza_id)
      ORDER BY order_id
   )
   SELECT SUM(COST + COALESCE(extra_cost, 0)) AS revenue FROM calc_prices;
 ````
| revenue |
| ------- |
| 142     |

***2B. Add cheese is $1 extra:***
````sql
    WITH calc_prices AS (
      SELECT order_id, customer_id, pizza_id, pizza_name, 
         (
               CASE
                  WHEN pizza_name = 'Meatlovers' THEN 12
                  WHEN pizza_name = 'Vegetarian' THEN 10
               END
         ) AS cost,
         CARDINALITY(array_positions(string_to_array( extras, ', ' ), '4')) as cheese_cost,
         array_length(string_to_array( extras, ', ' ), 1) AS extra_cost
      FROM delivered_orders
      JOIN pizza_names
      USING (pizza_id)
      ORDER BY order_id
   )
   SELECT SUM(cost + COALESCE(extra_cost, 0) + cheese_cost) AS revenue FROM calc_prices;
 ````
| revenue |
| ------- |
| 143     |
---
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
````sql
    create table ratings (order_id int, customer_id int, runner_id int, rating int);

    insert into ratings values
    		(1, 101, 1, 5),
    		(2, 101, 1, 4),
    		(3, 102, 1, 5),
    		(4, 103, 2, 3),
    		(5, 104, 3, 2),
    		(7, 105, 2, 4),
    		(8, 102, 2, 5),
    		(10, 104, 1, 4);
````

---
### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
* customer_id
* order_id
* runner_id
* rating
* order_time
* pickup_time
* Time between order and pickup
* Delivery duration
* Average speed
* Total number of pizzas

````sql
    CREATE TABLE successful_deliveries AS
    SELECT order_id, cot.customer_id, rot.runner_id, count(pizza_id) AS pizza_count, rating, 
      order_time,	pickup_time, (pickup_time - order_time) AS response_time, duration,
      round(distance / (duration::numeric/60), 2) AS average_speed
    FROM customer_orders_temp cot
    JOIN runner_orders_temp rot
    USING (order_id)
    JOIN ratings r
    USING (order_id)
    GROUP BY order_id, cot.customer_id, rot.runner_id, rating, order_time,
      pickup_time, duration, distance
    ORDER BY order_id;
````





    SELECT * FROM successful_deliveries;

   ![Successful Deliveries](https://i.ibb.co/X2RS0qP/successful-deliveries.png)

---


### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
````sql
    WITH revenue AS (
      SELECT pizza_id, pizza_name,
         (
            CASE
               WHEN pizza_name = 'Meatlovers' THEN 12
               WHEN pizza_name = 'Vegetarian' THEN 10
            END
         ) AS revenue
      FROM delivered_orders
      JOIN pizza_names
      USING (pizza_id)
   ),
   logistics AS (
      SELECT SUM(distance * 0.3) AS logistic_cost
      FROM (SELECT DISTINCT order_id, distance FROM delivered_orders) received_orders
   )
   SELECT ROUND(SUM(revenue) - SUM(logistic_cost)/COUNT(logistic_cost),2) AS profit
   FROM revenue, logistics;
 ````        

| profit |
| ------ |
| 94.44  |

---

