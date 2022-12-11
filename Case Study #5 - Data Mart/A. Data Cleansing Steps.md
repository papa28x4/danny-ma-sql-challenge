
# 🛒 Case Study #5 - Data Mart

## 🛠️ Solution - A. Data Cleansing Steps

    SET search_path = data_mart;
    
In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:
- Convert the `week_date` to a `DATE` format
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called `age_band` after the original segment column using the following mapping on the number inside the segment value
  
<img width="166" alt="image" src="https://user-images.githubusercontent.com/81607668/131438667-3b7f3da5-cabc-436d-a352-2022841fc6a2.png">
  
- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:  

| segment | demographic | 
| ------- | ----------- |
| C       | Couples     |
| F       | Families    |

- Ensure all `null` string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the sales value divided by transactions rounded to 2 decimal places for each record


**Answer**
````sql
    DROP TABLE IF EXISTS clean_weekly_sales;

    CREATE TABLE clean_weekly_sales AS
    SELECT 
        TO_DATE(week_date, 'dd/mm/yy') AS week_date,
        EXTRACT(WEEK FROM TO_DATE(week_date, 'dd/mm/yy')) AS week_number,
        EXTRACT(MONTH FROM TO_DATE(week_date, 'dd/mm/yy')) AS month_number,
        EXTRACT(YEAR FROM TO_DATE(week_date, 'dd/mm/yy')) AS calendar_year,
        region,
        platform,
        segment,
        (
            CASE
                WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
                WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
                WHEN RIGHT(segment, 1) in ('3', '4') THEN 'Retirees'
                ELSE 'unknown'
            END
        ) AS age_band,
        (
            CASE
                WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
                WHEN LEFT(segment, 1) = 'F' THEN 'Families'
                ELSE 'unknown'
            END
        ) AS demographic,
        transactions,
        sales,
        ROUND(sales::NUMERIC/transactions, 2) AS avg_transaction
    FROM weekly_sales;
````

````sql
    SELECT * FROM clean_weekly_sales
    LIMIT 10;
````
| week_date                | week_number | month_number | calendar_year | region | platform | segment | age_band     | demographic | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------ | -------- | ------- | ------------ | ----------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA   | Retail   | C3      | Retirees     | Couples     | 120631       | 3656163  | 30.31           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA   | Retail   | F1      | Young Adults | Families    | 31574        | 996575   | 31.56           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | USA    | Retail   | null    | unknown      | unknown     | 529151       | 16509610 | 31.20           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | EUROPE | Retail   | C1      | Young Adults | Couples     | 4517         | 141942   | 31.42           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA | Retail   | C2      | Middle Aged  | Couples     | 58046        | 1758388  | 30.29           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | CANADA | Shopify  | F2      | Middle Aged  | Families    | 1336         | 243878   | 182.54          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA | Shopify  | F3      | Retirees     | Families    | 2514         | 519502   | 206.64          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA   | Shopify  | F1      | Young Adults | Families    | 2158         | 371417   | 172.11          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA | Shopify  | F2      | Middle Aged  | Families    | 318          | 49557    | 155.84          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA | Retail   | C3      | Retirees     | Couples     | 111032       | 3888162  | 35.02           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8)
