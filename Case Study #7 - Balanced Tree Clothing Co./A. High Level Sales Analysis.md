# 👗 Case Study #7 - Balanced Tree Clothing Co.

## 🍝 Solution - A. High Level Sales Analysis

**Schema (PostgreSQL v13)**

    SET search_path = balanced_tree;


---
**1. What was the total quantity sold for all products?**
````sql
    SELECT SUM(qty) AS total_qty FROM sales;
````
| total_qty |
| --------- |
| 45216     |

---
**2. What is the total generated revenue for all products before discounts?**
````sql
    SET lc_monetary = 'en_US';

    SELECT CAST(SUM(qty * price) AS money) AS total_non_discounted_revenue FROM sales;
````
| total_non_discounted_revenue |
| ---------------------------- |
| $1,289,453.00                |

---
**3. What was the total discount amount for all products?**
````sql
    SELECT CAST(SUM(qty * price * discount::numeric/100) AS money) total_discount_amount FROM sales;
````
| total_discount_amount |
| --------------------- |
| $156,229.14           |

