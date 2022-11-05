# 🥑 Case Study #3 - Foodie-Fi

## 🎞 Solution - B. Data Analysis Questions

**Schema (PostgreSQL v13)**
    SET search_path = foodie_fi;
    
---
### 1. How many customers has Foodie-Fi ever had?
````sql
    SELECT COUNT(DISTINCT customer_id) AS unique_customer
    FROM subscriptions;
````
| unique_customer |
| --------------- |
| 1000            |

---
### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
````sql
   SELECT EXTRACT(MONTH FROM start_date) AS month_no, UPPER(TO_CHAR(start_date, 'month')) AS month_name,
     count(customer_id) AS customer_count 
    FROM subscriptions 
    WHERE plan_id = (SELECT plan_id FROM plans WHERE plan_name = 'trial' )
    GROUP BY month_name, month_no
    ORDER BY month_no;
````
| month_no | month_name | customer_count |
| -------- | ---------- | -------------- |
| 1        | JANUARY    | 88             |
| 2        | FEBRUARY   | 68             |
| 3        | MARCH      | 94             |
| 4        | APRIL      | 81             |
| 5        | MAY        | 88             |
| 6        | JUNE       | 79             |
| 7        | JULY       | 89             |
| 8        | AUGUST     | 88             |
| 9        | SEPTEMBER  | 87             |
| 10       | OCTOBER    | 79             |
| 11       | NOVEMBER   | 75             |
| 12       | DECEMBER   | 84             |

---
### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.
````sql
    SELECT plan_id, INITCAP(plan_name) AS plan_name, COUNT(start_date) AS events 
    FROM subscriptions
    JOIN plans
    USING (plan_id)
    WHERE EXTRACT(YEAR FROM start_date) > '2020'
    GROUP BY plan_name, plan_id
    ORDER BY plan_id;
````
| plan_id | plan_name     | events |
| ------- | ------------- | ------ |
| 1       | Basic Monthly | 8      |
| 2       | Pro Monthly   | 60     |
| 3       | Pro Annual    | 63     |
| 4       | Churn         | 71     |

**You may also want to see how this compares with the previous year (2020)**

````sql
SELECT plan_id, INITCAP(plan_name) AS plan_name, 
    	COUNT(CASE WHEN EXTRACT(YEAR FROM start_date) = '2020' THEN 1 END) AS events_2020, 
    	COUNT(CASE WHEN EXTRACT(YEAR FROM start_date) > '2020' THEN 1 END) AS events_2021
FROM subscriptions
JOIN plans
USING (plan_id)
GROUP BY plan_name, plan_id
ORDER BY plan_id;
````
| plan_id | plan_name     | events_2020 | events_2021 |
| ------- | ------------- | ----------- | ----------- |
| 0       | Trial         | 1000        | 0           |
| 1       | Basic Monthly | 538         | 8           |
| 2       | Pro Monthly   | 479         | 60          |
| 3       | Pro Annual    | 195         | 63          |
| 4       | Churn         | 236         | 71          |

- we should compare it with jan to april 2020
---
### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
````sql
    SELECT 
        COUNT(CASE WHEN plan_name = 'churn' THEN 1 END) AS customer_churn,
        COUNT(DISTINCT customer_id) AS total_customers,
        ROUND(
                COUNT(CASE WHEN plan_name = 'churn' THEN 1 END)::numeric / COUNT(DISTINCT customer_id) * 100, 1
             ) AS churn_rate
    FROM subscriptions
    JOIN plans
    using (plan_id);
````
| customer_churn | total_customers | churn_rate |
| -------------- | --------------- | ---------- |
| 307            | 1000            | 30.7       |

---
### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
````sql
    WITH ranking AS (
        SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY start_date)
        FROM subscriptions
    )
    SELECT COUNT(CASE WHEN plan_name = 'churn' AND rank = 2 THEN 1 END) AS churn_count,
    ROUND(
            COUNT(CASE WHEN plan_name = 'churn' AND rank = 2 THEN 1 END)::numeric 
            / COUNT(DISTINCT customer_id) * 100
         ) AS churn_rate
    FROM ranking
    JOIN plans
    USING (plan_id);
````
| churn_count | churn_rate |
| ----------- | ---------- |
| 92          | 9          |

---
### 6. What is the number and percentage of customer plans after their initial free trial?
````sql
     WITH ranking AS (
        SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY start_date)
        FROM subscriptions
    )
    SELECT plan_id, plan_name, COUNT(plan_id) AS conversions,
    ROUND(
        COUNT(plan_id)::NUMERIC/(SELECT COUNT(plan_id) from ranking where rank = 2) * 100
        , 1) AS conversion_percentage
    FROM ranking
    JOIN plans
    USING (plan_id)
    WHERE rank = 2
    GROUP BY plan_id, plan_name
    ORDER BY plan_id;
````
| plan_id | plan_name     | conversions | conversion_percentage |
| ------- | ------------- | ----------- | --------------------- |
| 1       | basic monthly | 546         | 54.6                  |
| 2       | pro monthly   | 325         | 32.5                  |
| 3       | pro annual    | 37          | 3.7                   |
| 4       | churn         | 92          | 9.2                   |

---
### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
````sql
     WITH ranking AS (
        SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY start_date DESC)
        FROM subscriptions
        WHERE start_date <= '2020-12-31'
    )
    SELECT plan_id, count(plan_id) AS customer_count,
        ROUND(
        COUNT(plan_id)::NUMERIC/(SELECT COUNT(customer_id) from ranking where rank = 1) * 100
        , 1) AS percentage
    FROM ranking
    WHERE rank = 1
    GROUP BY plan_id;
````
| plan_id | customer_count | percentage |
| ------- | -------------- | ---------- |
| 0       | 19             | 1.9        |
| 1       | 224            | 22.4       |
| 2       | 326            | 32.6       |
| 3       | 195            | 19.5       |
| 4       | 236            | 23.6       |

---
### 8. How many customers have upgraded to an annual plan in 2020?
````sql
    SELECT COUNT(DISTINCT customer_id) AS customer_count
    FROM subscriptions
    JOIN plans
    USING (plan_id)
    WHERE plan_name = 'pro annual'
        AND EXTRACT(YEAR FROM start_date) = '2020';
````
| customer_count |
| -------------- |
| 195            |

---
### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
````sql
    SELECT ROUND(AVG(s2.start_date - s1.start_date)) AS average_days
    FROM subscriptions s1
    JOIN subscriptions s2
    ON s1.customer_id = s2.customer_id
        AND s1.plan_id +  3 = s2.plan_id
    WHERE s2.plan_id = 3;
````
| average_days |
| ------------ |
| 105          |

---
### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
````sql
    WITH duration_table AS (
        SELECT s2.start_date - s1.start_date AS duration,
        WIDTH_BUCKET(s2.start_date - s1.start_date, 1, 360, 12) AS bin
        FROM subscriptions s1
        JOIN subscriptions s2
        ON s1.customer_id = s2.customer_id
            AND s1.plan_id +  3 = s2.plan_id
        WHERE s2.plan_id = 3
        ORDER BY duration 
    )
    SELECT concat((bin-1)*30+1,' - ',bin*30,' days') AS breakdown, 
        COUNT(bin) AS customers
    FROM duration_table
    GROUP BY bin;
````
| breakdown      | customers |
| -------------- | --------- |
| 1 - 30 days    | 49        |
| 31 - 60 days   | 24        |
| 61 - 90 days   | 34        |
| 91 - 120 days  | 35        |
| 121 - 150 days | 42        |
| 151 - 180 days | 36        |
| 181 - 210 days | 26        |
| 211 - 240 days | 4         |
| 241 - 270 days | 5         |
| 271 - 300 days | 1         |
| 301 - 330 days | 1         |
| 331 - 360 days | 1         |

---
### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
````sql
    SELECT COUNT(DISTINCT s1.customer_id) AS customer_count
    FROM subscriptions s1
    JOIN subscriptions s2
    ON s1.customer_id = s2.customer_id
        AND s1.plan_id - 1 = s2.plan_id
    WHERE s2.plan_id = 1
        AND (s2.start_date - s1.start_date) > 0
        AND EXTRACT(YEAR FROM s2.start_date) = '2020';
````
| customer_count |
| -------------- |
| 0              |



