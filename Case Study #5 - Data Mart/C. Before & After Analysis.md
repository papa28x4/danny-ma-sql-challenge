**Schema (PostgreSQL v13)**

    SET search_path = data_mart;

---
**1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?**
````sql
    WITH four_weeks_before AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_date BETWEEN (TO_DATE('2020-06-15', 'yy/mm/dd') - interval '4 weeks') AND 
        (TO_DATE('2020-06-15', 'yy/mm/dd') - interval '1 week')
    ),
    four_weeks_after AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_date BETWEEN TO_DATE('2020-06-15', 'yy/mm/dd')  AND 
        (TO_DATE('2020-06-15', 'yy/mm/dd') + interval '3 weeks') 
    ),
    summations AS (
        SELECT SUM(CASE WHEN week_date in (select * from four_weeks_before) THEN sales END) AS four_weeks_before,
            SUM(CASE WHEN week_date in (select * from four_weeks_after) THEN sales END) AS four_weeks_after
        FROM clean_weekly_sales
    )
    SELECT *,
        four_weeks_after - four_weeks_before AS variance,
        ROUND(100 * (four_weeks_after - four_weeks_before)::numeric/four_weeks_after, 2) AS percentage_change
    FROM summations;
````
| four_weeks_before | four_weeks_after | variance  | percentage_change |
| ----------------- | ---------------- | --------- | ----------------- |
| 2345878357        | 2318994169       | -26884188 | -1.16             |

---
2. What about the entire 12 weeks before and after?
````sql
    WITH twelve_weeks_before AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_date BETWEEN (TO_DATE('2020-06-15', 'yy/mm/dd') - interval '12 weeks') AND 
        (TO_DATE('2020-06-15', 'yy/mm/dd') - interval '1 week')
    ),
    twelve_weeks_after AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_date BETWEEN TO_DATE('2020-06-15', 'yy/mm/dd')  AND 
        (TO_DATE('2020-06-15', 'yy/mm/dd') + interval '11 weeks') 
    ),
    summations AS (
        SELECT SUM(CASE WHEN week_date in (select * from twelve_weeks_before) THEN sales END) AS twelve_weeks_before,
            SUM(CASE WHEN week_date in (select * from twelve_weeks_after) THEN sales END) AS twelve_weeks_after
        FROM clean_weekly_sales
    )
    SELECT *,
        twelve_weeks_after - twelve_weeks_before AS variance,
        ROUND(100 * (twelve_weeks_after - twelve_weeks_before)::numeric/twelve_weeks_after, 2) AS percentage_change
    FROM summations;
````
| twelve_weeks_before | twelve_weeks_after | variance   | percentage_change |
| ------------------- | ------------------ | ---------- | ----------------- |
| 7126273147          | 6973947753         | -152325394 | -2.18             |

---
3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

**a) 4 weeks**
````sql
    WITH cte AS (
        SELECT DISTINCT week_number FROM clean_weekly_sales
        WHERE week_date = '2020-06-15'
    ),
    four_weeks_before AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_number BETWEEN (SELECT * FROM cte) - 4 AND (SELECT * FROM cte) - 1
    ),
    four_weeks_after AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_number BETWEEN (SELECT * FROM cte) AND (SELECT * FROM cte) + 3
    ),
    summations AS (
        SELECT calendar_year, SUM(CASE WHEN week_date in (select * from four_weeks_before) THEN sales END) AS four_weeks_before,
        SUM(CASE WHEN week_date in (select * from four_weeks_after) THEN sales END) AS four_weeks_after
        FROM clean_weekly_sales
        GROUP BY calendar_year
    )
    SELECT *,
        four_weeks_after - four_weeks_before AS variance,
        ROUND(100 * (four_weeks_after - four_weeks_before)::numeric/four_weeks_after, 2) AS percentage_change
    FROM summations
    ORDER BY calendar_year;
````
| calendar_year | four_weeks_before | four_weeks_after | variance  | percentage_change |
| ------------- | ----------------- | ---------------- | --------- | ----------------- |
| 2018          | 2125140809        | 2129242914       | 4102105   | 0.19              |
| 2019          | 2249989796        | 2252326390       | 2336594   | 0.10              |
| 2020          | 2345878357        | 2318994169       | -26884188 | -1.16             |

---
**b) 12 weeks**
````sql
    WITH cte AS (
        SELECT DISTINCT week_number FROM clean_weekly_sales
        WHERE week_date = '2020-06-15'
    ),
    twelve_weeks_before AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_number BETWEEN (SELECT * FROM cte) - 12 AND (SELECT * FROM cte) - 1
    ),
    twelve_weeks_after AS (
        SELECT DISTINCT week_date
        FROM clean_weekly_sales
        WHERE week_number BETWEEN (SELECT * FROM cte) AND (SELECT * FROM cte) + 11
    ),
    summations AS (
        SELECT calendar_year, SUM(CASE WHEN week_date in (select * from twelve_weeks_before) THEN sales END) AS twelve_weeks_before,
        SUM(CASE WHEN week_date in (select * from twelve_weeks_after) THEN sales END) AS twelve_weeks_after
        FROM clean_weekly_sales
        GROUP BY calendar_year
    )
    SELECT *,
        twelve_weeks_after - twelve_weeks_before AS variance,
        ROUND(100 * (twelve_weeks_after - twelve_weeks_before)::numeric/twelve_weeks_after, 2) AS percentage_change
    FROM summations
    ORDER BY calendar_year;
````
| calendar_year | twelve_weeks_before | twelve_weeks_after | variance   | percentage_change |
| ------------- | ------------------- | ------------------ | ---------- | ----------------- |
| 2018          | 6396562317          | 6500818510         | 104256193  | 1.60              |
| 2019          | 6883386397          | 6862646103         | -20740294  | -0.30             |
| 2020          | 7126273147          | 6973947753         | -152325394 | -2.18             |

---
