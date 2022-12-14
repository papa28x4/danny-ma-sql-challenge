# 🥑 Case Study #3: Foodie-Fi

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Background](#Background)
- [Available Data](#available-data)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#case-study-questions)
- Solution
  - [A. Customer Journey](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/A.%20Customer%20Journey.md)
  - [B. Data Analysis Questions](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/B.%20Data%20Analysis%20Questions.md)
  - [C. Challenge Payment Question]
  - [D. Outside The Box Questions]

***

## Background
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

## Available Data
Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.

All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png)

**Table 1: plans**

Customers can choose which plans to join Foodie-Fi when they first sign up.

Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90

Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.

When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

| plan_id | plan_name     | price  |
| ------- | ------------- | ------ |
| 0       | trial         | 0.00   |
| 1       | basic monthly | 9.90   |
| 2       | pro monthly   | 19.90  |
| 3       | pro annual    | 199.00 |
| 4       | churn         |        |


**Table 2: subscriptions**

Customer subscriptions show the exact date where their specific plan_id starts.

If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.


| customer_id | plan_id | start_date               |
| ----------- | ------- | ------------------------ |
| 1           | 0       | 2020-08-01T00:00:00.000Z |
| 1           | 1       | 2020-08-08T00:00:00.000Z |
| 2           | 0       | 2020-09-20T00:00:00.000Z |
| 2           | 3       | 2020-09-27T00:00:00.000Z |
| 3           | 0       | 2020-01-13T00:00:00.000Z |
| 3           | 1       | 2020-01-20T00:00:00.000Z |
| 4           | 0       | 2020-01-17T00:00:00.000Z |
| 4           | 1       | 2020-01-24T00:00:00.000Z |
| 4           | 4       | 2020-04-21T00:00:00.000Z |
| 5           | 0       | 2020-08-03T00:00:00.000Z |
| 5           | 1       | 2020-08-10T00:00:00.000Z |
| 6           | 0       | 2020-12-23T00:00:00.000Z |
| 6           | 1       | 2020-12-30T00:00:00.000Z |
| 6           | 4       | 2021-02-26T00:00:00.000Z |
| 7           | 0       | 2020-02-05T00:00:00.000Z |
| 7           | 1       | 2020-02-12T00:00:00.000Z |
| 7           | 2       | 2020-05-22T00:00:00.000Z |
| 8           | 0       | 2020-06-11T00:00:00.000Z |
| 8           | 1       | 2020-06-18T00:00:00.000Z |
| 8           | 2       | 2020-08-03T00:00:00.000Z |
| 9           | 0       | 2020-12-07T00:00:00.000Z |
| 9           | 3       | 2020-12-14T00:00:00.000Z |
| 10          | 0       | 2020-09-19T00:00:00.000Z |
| 10          | 2       | 2020-09-26T00:00:00.000Z |
| 11          | 0       | 2020-11-19T00:00:00.000Z |
| 11          | 4       | 2020-11-26T00:00:00.000Z |
| 12          | 0       | 2020-09-22T00:00:00.000Z |
| 12          | 1       | 2020-09-29T00:00:00.000Z |
| 13          | 0       | 2020-12-15T00:00:00.000Z |
| 13          | 1       | 2020-12-22T00:00:00.000Z |
| 13          | 2       | 2021-03-29T00:00:00.000Z |
| 14          | 0       | 2020-09-22T00:00:00.000Z |
| 14          | 1       | 2020-09-29T00:00:00.000Z |
| 15          | 0       | 2020-03-17T00:00:00.000Z |
| 15          | 2       | 2020-03-24T00:00:00.000Z |
| 15          | 4       | 2020-04-29T00:00:00.000Z |
| 16          | 0       | 2020-05-31T00:00:00.000Z |
| 16          | 1       | 2020-06-07T00:00:00.000Z |
| 16          | 3       | 2020-10-21T00:00:00.000Z |
| 17          | 0       | 2020-07-27T00:00:00.000Z |
| 17          | 1       | 2020-08-03T00:00:00.000Z |
| 17          | 3       | 2020-12-11T00:00:00.000Z |
| 18          | 0       | 2020-07-06T00:00:00.000Z |
| 18          | 2       | 2020-07-13T00:00:00.000Z |
| 19          | 0       | 2020-06-22T00:00:00.000Z |
| 19          | 2       | 2020-06-29T00:00:00.000Z |
| 19          | 3       | 2020-08-29T00:00:00.000Z |

***

## Case Study Questions

### A. Customer Journey

View my solution [here](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/A.%20Customer%20Journey.md).
  
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

### B. Data Analysis Questions

View my solution [here](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%233%20-%20Foodie-Fi/B.%20Data%20Analysis%20Questions.md).
  
1. How many customers has Foodie-Fi ever had?
2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
6. What is the number and percentage of customer plans after their initial free trial?
7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
8. How many customers have upgraded to an annual plan in 2020?
9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

### C. Challenge Payment Question
  
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

***
