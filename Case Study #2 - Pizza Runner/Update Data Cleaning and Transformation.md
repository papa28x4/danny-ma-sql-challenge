# 🍕 Case Study #2 - Pizza Runner

## 🛠️ Data Cleaning & Transformation

**Schema (PostgreSQL v13)**

  SET search_path = pizza_runner;
      
  ### 🗄️ Table: customer_orders
  
````sql
  DROP TABLE IF EXISTS customer_orders_temp;
  CREATE TABLE customer_orders_temp AS
  SELECT 
	  order_id, 
	  customer_id, 
	  pizza_id, 
	  CASE
		  WHEN exclusions is null OR exclusions = 'null' THEN ''
		  ELSE exclusions
		  END AS exclusions,
	  CASE
		  WHEN extras is null OR extras = 'null' THEN ''
		  ELSE extras
		  END AS extras,
		order_time
   FROM customer_orders;
````
  
  ### 🗄️ Table: runner_orders
  
  ````sql
DROP TABLE IF EXISTS runner_orders_temp;
CREATE TABLE runner_orders_temp AS
SELECT 
	  order_id, 
	  runner_id,  
	  CASE
	      WHEN pickup_time = 'null' THEN NULL
	      ELSE pickup_time
	  END AS pickup_time,
	  CASE
	      WHEN distance = 'null' THEN NULL
	      WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	      ELSE distance 
	  END AS distance,
	  CASE
	      WHEN duration = 'null' THEN NULL
	      WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	      WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	      WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	      ELSE duration
	      END AS duration,
	  CASE
	      WHEN cancellation IS NULL or cancellation = 'null' THEN ''
	      ELSE cancellation
	  END AS cancellation
FROM runner_orders;
````
**Next, we proceed to change `pickup_time`, `distance` and `duration` columns to the correct data type.**
    
  ````sql
	ALTER TABLE runner_orders_temp 
	ALTER COLUMN pickup_time TYPE timestamp USING pickup_time::timestamp without time zone, 
	ALTER COLUMN distance TYPE numeric USING distance::numeric,
	ALTER COLUMN duration TYPE integer USING duration::integer;
  ````        
