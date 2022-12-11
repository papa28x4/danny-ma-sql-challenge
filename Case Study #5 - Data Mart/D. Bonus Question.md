# 🛒 Case Study #5 - Data Mart

## 🧼 Solution - D. Bonus Question.md

**Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?**
- `region`
- `platform`
- `age_band`
- `demographic`
- `customer_type`

***Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?***

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
    		SELECT region, SUM(CASE WHEN week_date in (select * from twelve_weeks_before) THEN sales END) AS twelve_weeks_before,
    			SUM(CASE WHEN week_date in (select * from twelve_weeks_after) THEN sales END) AS twelve_weeks_after
    		FROM clean_weekly_sales
    		GROUP BY region
    	)
    	SELECT *,
    		twelve_weeks_after - twelve_weeks_before AS variance,
    		ROUND(100 * (twelve_weeks_after - twelve_weeks_before)::numeric/twelve_weeks_after, 2) AS percentage_change
    	FROM summations
    	ORDER BY percentage_change;
````
| region        | twelve_weeks_before | twelve_weeks_after | variance  | percentage_change |
| ------------- | ------------------- | ------------------ | --------- | ----------------- |
| ASIA          | 1637244466          | 1583807621         | -53436845 | -3.37             |
| OCEANIA       | 2354116790          | 2282795690         | -71321100 | -3.12             |
| SOUTH AMERICA | 213036207           | 208452033          | -4584174  | -2.20             |
| CANADA        | 426438454           | 418264441          | -8174013  | -1.95             |
| USA           | 677013558           | 666198715          | -10814843 | -1.62             |
| AFRICA        | 1709537105          | 1700390294         | -9146811  | -0.54             |
| EUROPE        | 108886567           | 114038959          | 5152392   | 4.52              |
