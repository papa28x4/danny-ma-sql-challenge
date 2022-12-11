# 🍅 Case Study #8 - Fresh Segments

## 🧼 Solution - A. Data Exploration and Cleansing

**Schema (PostgreSQL v13)**

    SET search_path = fresh_segments;


---
### 1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a `date` data type with the start of the month
```sql
    ALTER TABLE interest_metrics
    ALTER month_year TYPE DATE USING to_date(month_year, 'MM-YY')::DATE;
 ```  
![fresh_segments table](https://i.ibb.co/cX1DH0t/fresh-segment-altered.png)

### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```sql
    SELECT month_year, COUNT(*) 
    FROM interest_metrics
    GROUP BY month_year
    ORDER BY month_year NULLS FIRST;
```
| month_year               | count |
| ------------------------ | ----- |
|                          | 1194  |
| 2018-07-01T00:00:00.000Z | 729   |
| 2018-08-01T00:00:00.000Z | 767   |
| 2018-09-01T00:00:00.000Z | 780   |
| 2018-10-01T00:00:00.000Z | 857   |
| 2018-11-01T00:00:00.000Z | 928   |
| 2018-12-01T00:00:00.000Z | 995   |
| 2019-01-01T00:00:00.000Z | 973   |
| 2019-02-01T00:00:00.000Z | 1121  |
| 2019-03-01T00:00:00.000Z | 1136  |
| 2019-04-01T00:00:00.000Z | 1099  |
| 2019-05-01T00:00:00.000Z | 857   |
| 2019-06-01T00:00:00.000Z | 824   |
| 2019-07-01T00:00:00.000Z | 864   |
| 2019-08-01T00:00:00.000Z | 1149  |

---
### 3.What do you think we should do with these null values in the fresh_segments.interest_metrics?
```sql
    SELECT  
        ROUND(100 * COUNT(*)::NUMERIC/(SELECT COUNT(*) FROM interest_metrics),2) AS null_perecentage
    FROM interest_metrics
    WHERE month_year IS NULL;
```
| null_perecentage |
| ---------------- |
| 8.37             |
```sql
 DELETE FROM interest_metrics
 WHERE month_year IS NULL;
 ```
---
### 4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?

   
```sql
    SELECT COUNT(DISTINCT m.id) AS map_interest_ids,
    		COUNT(DISTINCT i.interest_id) AS metric_interest_ids,
    		COUNT(CASE WHEN i.interest_id IS NULL THEN 1 END) AS in_map_not_in_metric,
    		COUNT(CASE WHEN m.id IS NULL THEN 1 END) AS in_metric_not_in_map
    FROM interest_metrics i
    FULL OUTER JOIN interest_map m
    ON i.interest_id::int = m.id;
```
| map_interest_ids | metric_interest_ids | in_map_not_in_metric | in_metric_not_in_map |
| ---------------- | ------------------- | -------------------- | -------------------- |
| 1209             | 1202                | 7                    | 0                    |

---

### 5. Summarise the id values in the `fresh_segments.interest_map` by its total record count in this table
```sql
    SELECT COUNT(id)
    FROM interest_map;
```
| count |
| ----- |
| 1209  |


---
### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where 'interest_id = 21246' in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the id column.

ALTER TABLE interest_metrics
ALTER COLUMN interest_id TYPE integer USING interest_id::integer;
```sql        
    SELECT * 
    FROM fresh_segments.interest_map map
    INNER JOIN fresh_segments.interest_metrics metrics
      ON map.id = metrics.interest_id
    WHERE metrics.interest_id = 21246   
      AND metrics._month IS NOT NULL;
```
| id    | interest_name                    | interest_summary                                      | created_at               | last_modified            | _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking |
| ----- | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ | ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 7      | 2018  | 2018-07-01T00:00:00.000Z | 21246       | 2.26        | 0.65        | 722     | 0.96               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 8      | 2018  | 2018-08-01T00:00:00.000Z | 21246       | 2.13        | 0.59        | 765     | 0.26               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 9      | 2018  | 2018-09-01T00:00:00.000Z | 21246       | 2.06        | 0.61        | 774     | 0.77               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 10     | 2018  | 2018-10-01T00:00:00.000Z | 21246       | 1.74        | 0.58        | 855     | 0.23               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 11     | 2018  | 2018-11-01T00:00:00.000Z | 21246       | 2.25        | 0.78        | 908     | 2.16               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 12     | 2018  | 2018-12-01T00:00:00.000Z | 21246       | 1.97        | 0.7         | 983     | 1.21               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 1      | 2019  | 2019-01-01T00:00:00.000Z | 21246       | 2.05        | 0.76        | 954     | 1.95               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 2      | 2019  | 2019-02-01T00:00:00.000Z | 21246       | 1.84        | 0.68        | 1109    | 1.07               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 3      | 2019  | 2019-03-01T00:00:00.000Z | 21246       | 1.75        | 0.67        | 1123    | 1.14               |
| 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z | 4      | 2019  | 2019-04-01T00:00:00.000Z | 21246       | 1.58        | 0.63        | 1092    | 0.64               |

---
### 7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?
```sql
    WITH cte AS (
    	 	 SELECT * 
    		 FROM interest_map map
    		 INNER JOIN interest_metrics metrics
    		  ON map.id = metrics.interest_id
    		 WHERE metrics.month_year < map.created_at::date
    	 )
    	 SELECT 
    	 	COUNT(CASE 
    		 	WHEN EXTRACT(MONTH FROM month_year) < EXTRACT(MONTH FROM created_at) OR
    				 EXTRACT(YEAR FROM month_year) < EXTRACT(YEAR FROM created_at)
    			THEN 1
    			ELSE NULL
    		 END)
    	 FROM cte;
```
| count |
| ----- |
| 0     |

---
