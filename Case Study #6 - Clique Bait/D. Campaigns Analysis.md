# 🐟 Case Study #6 - Clique Bait

## 👩🏻‍💻 Solution - D. Campaigns Analysis

**Schema (PostgreSQL v13)**

SET search_path = pizza_runner;

Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- `user_id`
- `visit_id`
- `visit_start_time`: the earliest event_time for each visit
- `page_views`: count of page views for each visit
- `cart_adds`: count of product cart add events for each visit
- `purchase`: 1/0 flag if a purchase event exists for each visit
- `campaign_name`: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- `impression`: count of ad impressions for each visit
- `click`: count of ad clicks for each visit
- (Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
   
 ````sql
 WITH cte AS (
      SELECT user_id, visit_id, sequence_number, event_time AS visit_start_time
      FROM events e 
      JOIN users u
      USING (cookie_id)
      WHERE sequence_number = 1 
      ORDER BY visit_id
   ),
   cte2 AS (
      SELECT cte.*, start_date, end_date, campaign_name
      FROM cte
      LEFT JOIN campaign_identifier
         ON visit_start_time BETWEEN start_date AND end_date
   ),
   cte3 AS (
      SELECT visit_id, COUNT(CASE WHEN event_type = 1 THEN 1 END) AS page_views,
      COUNT(CASE WHEN event_type = 2 THEN 1 END) AS cart_adds,
      MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchased,
      COUNT(CASE WHEN event_type = 4 THEN 1 END) AS ad_impressions,
      COUNT(CASE WHEN event_type = 5 THEN 1 END) AS ad_clicks,
      STRING_AGG(CASE WHEN event_type = 2 THEN page_name ELSE NULL END, ', ' ORDER BY sequence_number) AS cart_products
      FROM events
      JOIN page_hierarchy
      USING (page_id)
      GROUP BY visit_id
   )
   SELECT c.user_id, c.visit_id, c.visit_start_time, page_views, cart_adds, purchased, 
      ad_impressions, ad_clicks, campaign_name, cart_products FROM cte
   JOIN cte2 c
   USING (visit_id)
   JOIN cte3
   USING (visit_id)
   ORDER BY user_id
   LIMIT 20;
 ````
 
| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchased | ad_impressions | ad_clicks | campaign_name                     | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | --------- | --------- | -------------- | --------- | --------------------------------- | ------------------------------------------------------------------------------------- |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3         | 1         | 0              | 0         | Half Off - Treat Your Shellf(ish) | Lobster, Crab, Oyster                                                                 |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3         | 1         | 0              | 0         | Half Off - Treat Your Shellf(ish) | Russian Caviar, Crab, Oyster                                                          |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) | Lobster                                                                               |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) |                                                                                       |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7         | 1         | 1              | 1         | Half Off - Treat Your Shellf(ish) | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8         | 1         | 1              | 1         | Half Off - Treat Your Shellf(ish) | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6         | 1         | 1              | 1         | Half Off - Treat Your Shellf(ish) | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) |                                                                                       |
| 2       | 1f1198   | 2020-02-01T21:51:55.078Z | 1          | 0         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) |                                                                                       |
| 2       | 0635fb   | 2020-02-16T06:42:42.735Z | 9          | 4         | 1         | 0              | 0         | Half Off - Treat Your Shellf(ish) | Salmon, Kingfish, Abalone, Crab                                                       |
| 2       | 910d9a   | 2020-02-01T10:40:46.875Z | 8          | 1         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) | Abalone                                                                               |
| 2       | 49d73d   | 2020-02-16T06:21:27.138Z | 11         | 9         | 1         | 1              | 1         | Half Off - Treat Your Shellf(ish) | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
| 2       | 3b5871   | 2020-01-18T10:16:32.158Z | 9          | 6         | 1         | 1              | 1         | 25% Off - Living The Lux Life     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | e26a84   | 2020-01-18T16:06:40.907Z | 6          | 2         | 1         | 0              | 0         | 25% Off - Living The Lux Life     | Salmon, Oyster                                                                        |
| 2       | c5c0ee   | 2020-01-18T10:35:22.765Z | 1          | 0         | 0         | 0              | 0         | 25% Off - Living The Lux Life     |                                                                                       |
| 2       | d58cbd   | 2020-01-18T23:40:54.761Z | 8          | 4         | 0         | 0              | 0         | 25% Off - Living The Lux Life     | Kingfish, Tuna, Abalone, Crab                                                         |
| 3       | 76ee84   | 2020-05-28T20:11:54.997Z | 7          | 3         | 1         | 0              | 0         |                                   | Salmon, Lobster, Crab                                                                 |
| 3       | dda9ae   | 2020-04-08T18:24:44.859Z | 10         | 8         | 1         | 1              | 1         |                                   | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 3       | 7e89a0   | 2020-05-28T10:57:51.749Z | 9          | 6         | 0         | 1              | 1         |                                   | Salmon, Tuna, Russian Caviar, Black Truffle, Lobster, Crab                            |
| 3       | 25502e   | 2020-02-21T11:26:15.353Z | 1          | 0         | 0         | 0              | 0         | Half Off - Treat Your Shellf(ish) |                 
