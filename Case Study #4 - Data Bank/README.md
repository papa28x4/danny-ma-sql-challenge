## Case Study #4: Data Bank

<img src="https://user-images.githubusercontent.com/81607668/130343294-a8dcceb7-b6c3-4006-8ad2-fab2f6905258.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Background](#background)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#case-study-questions)
- Solution
  - [A. Customer Nodes Exploration](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/A.%20Customer%20Nodes%20Exploration.md)
  - [B. Customer Transactions](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/B.%20Customer%20Transactions.md)
 
***

## Background
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## Entity Relationship Diagram

<img width="631" alt="image" src="https://user-images.githubusercontent.com/81607668/130343339-8c9ff915-c88c-4942-9175-9999da78542c.png">

**Table 1: Regions**

This regions table contains the region_id and their respective region_name values.

| region_id | region_name |
| --------- | ----------- |
| 1         | Australia   |
| 2         | America     |
| 3         | Africa      |
| 4         | Asia        |
| 5         | Europe      |


**Table 2: Customer Nodes**

Customers are randomly distributed across the nodes according to their region. This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

| customer_id | region_id | node_id | start_date               | end_date                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z |
| 2           | 3         | 5       | 2020-01-03T00:00:00.000Z | 2020-01-17T00:00:00.000Z |
| 3           | 5         | 4       | 2020-01-27T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 4           | 5         | 4       | 2020-01-07T00:00:00.000Z | 2020-01-19T00:00:00.000Z |
| 5           | 3         | 3       | 2020-01-15T00:00:00.000Z | 2020-01-23T00:00:00.000Z |
| 6           | 1         | 1       | 2020-01-11T00:00:00.000Z | 2020-02-06T00:00:00.000Z |
| 7           | 2         | 5       | 2020-01-20T00:00:00.000Z | 2020-02-04T00:00:00.000Z |
| 8           | 1         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 9           | 4         | 5       | 2020-01-21T00:00:00.000Z | 2020-01-25T00:00:00.000Z |
| 10          | 3         | 4       | 2020-01-13T00:00:00.000Z | 2020-01-14T00:00:00.000Z |


**Table 3: Customer Transactions**

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

| customer_id | txn_date                 | txn_type | txn_amount |
| ----------- | ------------------------ | -------- | ---------- |
| 429         | 2020-01-21T00:00:00.000Z | deposit  | 82         |
| 155         | 2020-01-10T00:00:00.000Z | deposit  | 712        |
| 398         | 2020-01-01T00:00:00.000Z | deposit  | 196        |
| 255         | 2020-01-14T00:00:00.000Z | deposit  | 563        |
| 185         | 2020-01-29T00:00:00.000Z | deposit  | 626        |
| 309         | 2020-01-13T00:00:00.000Z | deposit  | 995        |
| 312         | 2020-01-20T00:00:00.000Z | deposit  | 485        |
| 376         | 2020-01-03T00:00:00.000Z | deposit  | 706        |
| 188         | 2020-01-13T00:00:00.000Z | deposit  | 601        |
| 138         | 2020-01-11T00:00:00.000Z | deposit  | 520        |
***

## Case Study Questions

### A. Customer Nodes Exploration

View my solution [here](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/A.%20Customer%20Nodes%20Exploration.md).

1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

### B. Customer Transactions

View my solution [here](https://github.com/papa28x4/danny-ma-sql-challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/B.%20Customer%20Transactions.md).
  
1. What is the unique count and total amount for each transaction type?
2. What is the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers:
  - Have a negative first month balance?
  - Have a positive first month balance?
  - Increase their opening month’s positive closing balance by more than 5% in the following month?
  - Reduce their opening month’s positive closing balance by more than 5% in the following month?
  - Move from a positive balance in the first month to a negative balance in the second month?
  
***

Do give me a 🌟 if you like what you're reading. Thank you! 🙆🏻‍♀️
