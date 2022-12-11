# 🛒 Case Study #5 - Data Mart

## 🛍 Solution - B. Data Exploration


    SET search_path = data_mart;



---
**1. What day of the week is used for each week_date value?**
````sql
    SELECT DISTINCT TO_CHAR(week_date, 'day') AS week_day 
    FROM clean_weekly_sales;
````
| week_day  |
| --------- |
| monday    |

---
**2. What range of week numbers are missing from the dataset?**
````sql
    CREATE VIEW weeks_in_year AS
    SELECT GENERATE_SERIES(1,52) AS week_number;
````
***Method A***
````sql
    SELECT wiy.week_number 
    FROM weeks_in_year wiy 
    WHERE wiy.week_number NOT IN (SELECT DISTINCT cws.week_number FROM clean_weekly_sales cws) 
    ORDER BY week_number;
````
***Method B***
````sql
    SELECT wiy.week_number 
    FROM weeks_in_year wiy 
    LEFT JOIN clean_weekly_sales cws
    ON wiy.week_number = cws.week_number
    WHERE cws.week_number is null
    ORDER BY week_number;
````
| week_number |
| ----------- |
| 1           |
| 2           |
| 3           |
| 4           |
| 5           |
| 6           |
| 7           |
| 8           |
| 9           |
| 10          |
| 11          |
| 12          |
| 37          |
| 38          |
| 39          |
| 40          |
| 41          |
| 42          |
| 43          |
| 44          |
| 45          |
| 46          |
| 47          |
| 48          |
| 49          |
| 50          |
| 51          |
| 52          |

---
**3. How many total transactions were there for each year in the dataset?**
````sql
    SELECT calendar_year, sum(transactions) AS total_transactions
    FROM clean_weekly_sales
    GROUP BY calendar_year;
````
| calendar_year | total_transactions |
| ------------- | ------------------ |
| 2019          | 365639285          |
| 2018          | 346406460          |
| 2020          | 375813651          |

---
**4. What is the total sales for each region for each month?**
````sql
    SELECT region, TO_CHAR(TO_DATE(month_number::varchar, 'MM'), 'Month') AS month, 
        sum(sales) AS total
    FROM clean_weekly_sales
    GROUP BY region, month_number
    ORDER BY region, month_number;
````
| region        | month     | total      |
| ------------- | --------- | ---------- |
| AFRICA        | March     | 567767480  |
| AFRICA        | April     | 1911783504 |
| AFRICA        | May       | 1647244738 |
| AFRICA        | June      | 1767559760 |
| AFRICA        | July      | 1960219710 |
| AFRICA        | August    | 1809596890 |
| AFRICA        | September | 276320987  |
| ASIA          | March     | 529770793  |
| ASIA          | April     | 1804628707 |
| ASIA          | May       | 1526285399 |
| ASIA          | June      | 1619482889 |
| ASIA          | July      | 1768844756 |
| ASIA          | August    | 1663320609 |
| ASIA          | September | 252836807  |
| CANADA        | March     | 144634329  |
| CANADA        | April     | 484552594  |
| CANADA        | May       | 412378365  |
| CANADA        | June      | 443846698  |
| CANADA        | July      | 477134947  |
| CANADA        | August    | 447073019  |
| CANADA        | September | 69067959   |
| EUROPE        | March     | 35337093   |
| EUROPE        | April     | 127334255  |
| EUROPE        | May       | 109338389  |
| EUROPE        | June      | 122813826  |
| EUROPE        | July      | 136757466  |
| EUROPE        | August    | 122102995  |
| EUROPE        | September | 18877433   |
| OCEANIA       | March     | 783282888  |
| OCEANIA       | April     | 2599767620 |
| OCEANIA       | May       | 2215657304 |
| OCEANIA       | June      | 2371884744 |
| OCEANIA       | July      | 2563459400 |
| OCEANIA       | August    | 2432313652 |
| OCEANIA       | September | 372465518  |
| SOUTH AMERICA | March     | 71023109   |
| SOUTH AMERICA | April     | 238451531  |
| SOUTH AMERICA | May       | 201391809  |
| SOUTH AMERICA | June      | 218247455  |
| SOUTH AMERICA | July      | 235582776  |
| SOUTH AMERICA | August    | 221166052  |
| SOUTH AMERICA | September | 34175583   |
| USA           | March     | 225353043  |
| USA           | April     | 759786323  |
| USA           | May       | 655967121  |
| USA           | June      | 703878990  |
| USA           | July      | 760331754  |
| USA           | August    | 712002790  |
| USA           | September | 110532368  |

---
**5. What is the total count of transactions for each platform?**
````sql
    SELECT platform, sum(transactions) AS total_transactions
    FROM clean_weekly_sales
    GROUP BY platform;
````
| platform | total_transactions |
| -------- | ------------------ |
| Shopify  | 5925169            |
| Retail   | 1081934227         |

---
**6. What is the percentage of sales for Retail vs Shopify for each month?**
````sql
    SELECT calendar_year, month_number,
        ROUND(100 * SUM(CASE
                WHEN platform = 'Retail' THEN sales::numeric 
            END)/SUM(sales),2)  AS retail_percentage,
        ROUND(100 * SUM(CASE
                WHEN platform = 'Shopify' THEN sales::numeric 
            END)/SUM(sales), 2) AS shopify_percentage
    FROM clean_weekly_sales
    GROUP BY calendar_year, month_number
    ORDER BY calendar_year, month_number;
````
| calendar_year | month_number | retail_percentage | shopify_percentage |
| ------------- | ------------ | ----------------- | ------------------ |
| 2018          | 3            | 97.92             | 2.08               |
| 2018          | 4            | 97.93             | 2.07               |
| 2018          | 5            | 97.73             | 2.27               |
| 2018          | 6            | 97.76             | 2.24               |
| 2018          | 7            | 97.75             | 2.25               |
| 2018          | 8            | 97.71             | 2.29               |
| 2018          | 9            | 97.68             | 2.32               |
| 2019          | 3            | 97.71             | 2.29               |
| 2019          | 4            | 97.80             | 2.20               |
| 2019          | 5            | 97.52             | 2.48               |
| 2019          | 6            | 97.42             | 2.58               |
| 2019          | 7            | 97.35             | 2.65               |
| 2019          | 8            | 97.21             | 2.79               |
| 2019          | 9            | 97.09             | 2.91               |
| 2020          | 3            | 97.30             | 2.70               |
| 2020          | 4            | 96.96             | 3.04               |
| 2020          | 5            | 96.71             | 3.29               |
| 2020          | 6            | 96.80             | 3.20               |
| 2020          | 7            | 96.67             | 3.33               |
| 2020          | 8            | 96.51             | 3.49               |

---
**7. What is the percentage of sales by demographic for each year in the dataset?**
````sql
    SELECT calendar_year,
        ROUND(100 * SUM(CASE
                WHEN demographic = 'Couples' THEN sales::numeric 
            END)/SUM(sales),2)  AS couples_sales,
        ROUND(100 * SUM(CASE
                WHEN demographic = 'Families' THEN sales::numeric 
            END)/SUM(sales), 2) AS families_sales,
        ROUND(100 * SUM(CASE
                WHEN demographic = 'unknown' THEN sales::numeric 
            END)/SUM(sales), 2) AS unknown_sales
    FROM clean_weekly_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;
````
| calendar_year | couples_sales | families_sales | unknown_sales |
| ------------- | ------------- | -------------- | ------------- |
| 2018          | 26.38         | 31.99          | 41.63         |
| 2019          | 27.28         | 32.47          | 40.25         |
| 2020          | 28.72         | 32.73          | 38.55         |

---
**8. Which age_band and demographic values contribute the most to Retail sales?**
````sql
    WITH retail_total as (
        SELECT * 
        FROM clean_weekly_sales
        WHERE platform = 'Retail'
    )
    SELECT age_band, demographic, 
        sum(sales) AS subtotal,
        ROUND(100 * sum(sales)::NUMERIC / (SELECT sum(sales) FROM retail_total), 2) AS percentage_contribution
    FROM retail_total
    GROUP BY age_band, demographic
    ORDER BY subtotal DESC;
````
| age_band     | demographic | subtotal    | percentage_contribution |
| ------------ | ----------- | ----------- | ----------------------- |
| unknown      | unknown     | 16067285533 | 40.52                   |
| Retirees     | Families    | 6634686916  | 16.73                   |
| Retirees     | Couples     | 6370580014  | 16.07                   |
| Middle Aged  | Families    | 4354091554  | 10.98                   |
| Young Adults | Couples     | 2602922797  | 6.56                    |
| Middle Aged  | Couples     | 1854160330  | 4.68                    |
| Young Adults | Families    | 1770889293  | 4.47                    |

---
**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**
````sql
    SELECT calendar_year, platform, 
        ROUND(AVG(avg_transaction), 2) AS incorrect_avg_transactions,
        ROUND(SUM(sales)::NUMERIC/SUM(transactions), 2) AS proper_avg_transactions
    FROM clean_weekly_sales
    GROUP BY calendar_year, platform 
    ORDER BY calendar_year, platform;
````
| calendar_year | platform | incorrect_avg_transactions | proper_avg_transactions |
| ------------- | -------- | -------------------------- | ----------------------- |
| 2018          | Retail   | 42.91                      | 36.56                   |
| 2018          | Shopify  | 188.28                     | 192.48                  |
| 2019          | Retail   | 41.97                      | 36.83                   |
| 2019          | Shopify  | 177.56                     | 183.36                  |
| 2020          | Retail   | 40.64                      | 36.56                   |
| 2020          | Shopify  | 174.87                     | 179.03                  |

---
