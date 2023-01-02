# 🥑 Case Study #3 - Foodie-Fi

## 🎞 Solution - A. Customer Journey

**WIP**
Based off the 8 sample customers provided in the sample subscriptions table below, write a brief description about each customer’s onboarding journey.

<img width="261" alt="Screenshot 2021-08-17 at 11 36 10 PM" src="https://user-images.githubusercontent.com/81607668/129756709-75919d79-e1cd-4187-a129-bdf90a65e196.png">

**Answer:**

````sql
SELECT
  s.customer_id,f.plan_id, f.plan_name,  s.start_date
FROM foodie_fi.plans f
JOIN foodie_fi.subscriptions s
  ON f.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
````

<img width="556" alt="image" src="https://user-images.githubusercontent.com/81607668/129758340-b7cd527c-31f3-4f33-8d99-5b0a4baab378.png">

From the sample results, I will choose 3 customers and write about their onboarding journey.
