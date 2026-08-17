## Case Study #1 - Danny's Diner

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

Questions & Solutions

1. What is the total amount each customer spent at the restaurant?

Answer :

```sql
SELECT
customer_id,
SUM(price) as total_price
FROM (
    SELECT s.customer_id,
    s.product_id,
    m.price
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON m.product_id =s.product_id)t
GROUP BY customer_id
ORDER BY customer_id
```
| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

Customer A spent 76$, Customer B spent 74$ and Customer C spent 36$.

---

2. How many days has each customer visited the restaurant?
```sql    
SELECT
customer_id,
COUNT (DISTINCT order_date) as visits
FROM dannys_diner.sales as s
GROUP BY customer_id
```
| customer_id | visits |
| ----------- | ------ |
| A           | 4      |
| B           | 6      |
| C           | 2      |

Customer A visited 4 times, whereas Customer B visited 6 times and lastly Customer C visited 2 times.

---
3. What was the first item from the menu purchased by each customer?
```sql   
SELECT customer_id,
product_name
FROM (
    SELECT s.customer_id,
    RANK () OVER (PARTITION BY s.customer_id ORDER BY s.order_date) as rank,
    m.product_name
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON m.product_id =s.product_id)t
WHERE rank = 1
GROUP BY customer_id, product_name
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql 
SELECT
m.product_name,
COUNT(m.product_name) as most_purchase
FROM dannys_diner.sales as s
LEFT JOIN dannys_diner.menu as m
ON m.product_id =s.product_id
GROUP BY m.product_name
LIMIT 1;
```
| product_name | most_purchase |
| ------------ | ------------- |
| ramen        | 8             |

The highest purchased item in the menu is ramen which is 8 times.

---
5. Which item was the most popular for each customer?
```sql 
SELECT 
customer_id,
product_name,
total_count
FROM (
    SELECT
    s.customer_id,
    m.product_name,
    COUNT(m.product_name) AS total_count,
    RANK () OVER (PARTITION BY s.customer_id ORDER BY COUNT(m.product_name)DESC) AS rank
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON m.product_id =s.product_id
    GROUP BY s.customer_id, m.product_name
    ORDER BY s.customer_id)t
 WHERE rank =1
```
| customer_id | product_name | total_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

Customer A and C top product menu is Ramen, whereas Customer B enjoys all menu.

---
6.  Which item was purchased first by the customer after they became a member?

```sql
SELECT
customer_id,
product_name
FROM (
    SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    mem.join_date,
    DENSE_RANK () OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id
    LEFT JOIN dannys_diner.members as mem
    ON s.customer_id = mem.customer_id
    WHERE s.order_date > mem.join_date)t
WHERE rank=1
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

---

7. Which item was purchased just before the customer became a member?
```sql 
SELECT 
customer_id,
product_name
FROM (
    SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    ROW_NUMBER () OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id
    LEFT JOIN dannys_diner.members as mem
    ON s.customer_id = mem.customer_id
    WHERE s.order_date < mem.join_date)t
WHERE rank = 1
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

---

8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT *
FROM (
    SELECT
    s.customer_id,
    COUNT (s.product_id) OVER (PARTITION BY s.customer_id) AS total_item,
    SUM (m.price) OVER (PARTITION BY s.customer_id) AS total_price
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id
    LEFT JOIN dannys_diner.members as mem
    ON s.customer_id = mem.customer_id
    WHERE s.order_date < mem.join_date)t
GROUP BY customer_id, total_price, total_item
ORDER BY customer_id
```
| customer_id | total_item | total_price |
| ----------- | ---------- | ----------- |
| A           | 2          | 25          |
| B           | 3          | 40          |

---

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT
customer_id,
SUM (CASE
    WHEN product_name = 'sushi' THEN total * 20
    ELSE total * 10
END)as points
FROM (
    SELECT
    s.customer_id,
    m.product_name,
    m.price,
    COUNT(m.product_name) as total_item,
    COUNT(m.product_name) * m.price as total
    FROM dannys_diner.sales as s
    LEFT JOIN dannys_diner.menu as m
    ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name, m.price
    ORDER BY customer_id)t
GROUP BY t.customer_id
```
| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

