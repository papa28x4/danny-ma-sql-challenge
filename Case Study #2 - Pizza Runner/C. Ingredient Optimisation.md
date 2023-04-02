# 🍕 Case Study #2 Pizza Runner

## Solution - C. Ingredient Optimisation

**Schema (PostgreSQL v13)**

    SET search_path = pizza_runner;
    
---
### 1. What are the standard ingredients for each pizza?

````sql
    SELECT  pizza_name, string_agg(pt.topping_name, ', ') AS ingredients
    FROM    pizza_recipes pr
            JOIN pizza_toppings pt
                ON pt.topping_id::varchar = any( string_to_array( pr.toppings, ', ' ) )
            JOIN pizza_names
                using (pizza_id)
    GROUP   BY pizza_name;
````

| pizza_name | ingredients                                                           |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

---
### 2. What was the most commonly added extra?

````sql
    SELECT *,
        (
            SELECT COUNT(extras) FROM customer_orders_temp
            WHERE topping_id::varchar = ANY( string_to_array( extras, ', ' ) )
        ) AS extra_count
    FROM pizza_toppings
    ORDER BY extra_count DESC
    LIMIT 1;
````

| topping_id | topping_name | extra_count |
| ---------- | ------------ | ----------- |
| 1          | Bacon        | 4           |

---
### 3. What was the most common exclusion?

````sql
    SELECT *,
        (
            SELECT COUNT(exclusions) FROM customer_orders_temp
            WHERE topping_id::varchar = ANY( string_to_array( exclusions, ', ' ) )
        ) AS exclusions_count
    FROM pizza_toppings
    ORDER BY exclusions_count DESC
    LIMIT 1;
````

| topping_id | topping_name | exclusions_count |
| ---------- | ------------ | ---------------- |
| 4          | Cheese       | 4                |

---
### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:

````sql
    WITH orders AS (
            SELECT  pizza_name, 
                (
                    CASE
                        WHEN exclusions = '' THEN exclusions
                        ELSE concat('- Exclude ' , ( select string_agg(topping_name, ', ') from pizza_toppings pt
                              where topping_id::varchar = any( string_to_array( cot.exclusions , ', ' ) )) )
                    END
                )AS exclusions, 
                (
                    CASE
                        WHEN extras = '' THEN extras
                        ELSE concat('- Extra ' , (select string_agg(topping_name, ', ') from pizza_toppings pt
                        where topping_id::varchar = any( string_to_array( cot.extras , ', ' ) ) ))
                    END
                )AS extras
            FROM customer_orders_temp cot
            JOIN pizza_names pn
            USING (pizza_id)
    )
    SELECT FORMAT('%s %s %s', pizza_name, exclusions, extras) order_list 
    FROM orders;
````

| order_list                                                      |
| --------------------------------------------------------------- |
| Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |
| Meatlovers                                                      |
| Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| Meatlovers                                                      |
| Meatlovers  - Extra Bacon                                       |
| Meatlovers - Exclude Cheese                                     |
| Meatlovers - Exclude Cheese                                     |
| Meatlovers                                                      |
| Meatlovers                                                      |
| Meatlovers                                                      |
| Vegetarian  - Extra Bacon                                       |
| Vegetarian                                                      |
| Vegetarian - Exclude Cheese                                     |
| Vegetarian                                                      |

---
### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
````sql
    CREATE OR REPLACE FUNCTION array_diff(array1 anyarray, array2 anyarray)
        RETURNS anyarray
        AS 
       '
        BEGIN
            return (select coalesce(array_agg(elem), $${}$$)
            from unnest(array1) elem
            where elem <> all(array2));
        END;
        ' 
     LANGUAGE plpgsql;
````
````sql
    CREATE OR REPLACE FUNCTION array_sort(array1 anyarray)
        RETURNS anyarray
        AS 
       '
        BEGIN
            RETURN (SELECT ARRAY(SELECT unnest($1) ORDER BY 1));
        END;
        ' 
    LANGUAGE plpgsql;
````
````sql
    CREATE VIEW customer_orders_view AS
    SELECT *, ROW_NUMBER() OVER() AS s_no 
    FROM customer_orders_temp;
````
````sql
    WITH cte AS (
        SELECT  order_id, pizza_id, pizza_name, string_agg(pt.topping_id::varchar, ', ') AS ingredients,
            exclusions, extras, s_no
        FROM pizza_recipes pr
        JOIN pizza_toppings pt
            ON pt.topping_id::varchar = any( string_to_array( pr.toppings, ', ' ) )
        JOIN pizza_names
            USING (pizza_id)
        JOIN customer_orders_view cov
            USING (pizza_id)
        GROUP   BY pr.pizza_id, pizza_name, order_id, exclusions, extras, s_no
    ),
    cte2 AS (
        SELECT pizza_id, pizza_name, s_no,
            ARRAY_DIFF((string_to_array( ingredients, ', ' ) || string_to_array( extras, ', ' )), 
            string_to_array(exclusions, ', ' )) AS list
        FROM cte
    ),
    cte3 AS (
        SELECT pizza_id, pizza_name, s_no, unnest(list) AS list 
        FROM cte2
    ),
    cte4 AS (
        SELECT  pizza_name, pt.topping_name AS ingredient, s_no
        FROM cte3
        LEFT JOIN pizza_toppings pt
            ON pt.topping_id::varchar = list
        ORDER BY s_no DESC
    ),   		
    cte5 AS (
        SELECT pizza_name, ingredient, count(ingredient) AS ingredient_count, s_no 
        FROM cte4
        GROUP BY pizza_name, ingredient, s_no
        ORDER BY s_no
    ),	
    cte6 AS (
        SELECT *, 
            (
                CASE
                    WHEN ingredient_count > 1 THEN concat(ingredient_count,'x',ingredient)
                    ELSE ingredient
                END
            ) AS modified_list
        FROM cte5
    ),
    cte7 AS (
        SELECT  pizza_name, 
            ARRAY_SORT(string_to_array(string_agg(modified_list, ' , '), ' , ')) AS ingredients, 
            s_no
        FROM cte6
        GROUP BY pizza_name, s_no
    )
    SELECT FORMAT('%s: %s', pizza_name, array_to_string(ingredients, ', ')) ingredient_list
    FROM cte7;
````
| ingredient_list                                                                     |
| ----------------------------------------------------------------------------------- |
| Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| Meatlovers: 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| Meatlovers: 2xBacon, 2xChicken, BBQ Sauce, Beef, Mushrooms, Pepperoni, Salami       |
| Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| Meatlovers: 2xBacon, 2xCheese, Beef, Chicken, Pepperoni, Salami                     |
| Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                      |
| Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes       |

---


### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

````sql
    WITH std_ingredients AS (
        SELECT pizza_id, toppings FROM delivered_orders
        JOIN pizza_recipes
        USING (pizza_id)
    ),
    total_list AS (
        SELECT topping_id, topping_name,
            (
                SELECT COUNT(1) FROM std_ingredients
                WHERE topping_id::varchar = ANY( string_to_array( toppings, ', ' ) )
            ) AS standard_count,

            (
                SELECT COUNT(1) FROM delivered_orders
                WHERE topping_id::varchar = ANY( string_to_array( exclusions, ', ' ) )
            ) AS exclusions_count,
            (
                SELECT COUNT(1) FROM delivered_orders
                WHERE topping_id::varchar = ANY( string_to_array( extras, ', ' ) )
            ) AS extras_count
        FROM pizza_toppings
    )
    SELECT topping_id, topping_name, (standard_count + extras_count - exclusions_count) AS total_qty
    FROM total_list
    ORDER BY total_qty DESC;
````

| topping_id | topping_name | total_qty |
| ---------- | ------------ | --------- |
| 1          | Bacon        | 12        |
| 6          | Mushrooms    | 11        |
| 5          | Cheese       | 10        |
| 8          | Pepperoni    | 9         |
| 10         | Salami       | 9         |
| 4          | Chicken      | 9         |
| 3          | Beef         | 9         |
| 2          | BBQ Sauce    | 8         |
| 12         | Tomato Sauce | 3         |
| 7          | Onions       | 3         |
| 9          | Peppers      | 3         |
| 11         | Tomatoes     | 3         |
