## Case Study #1 - Danny's Diner
https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138

Question & Solutions

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

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
How many days has each customer visited the restaurant?
What was the first item from the menu purchased by each customer?
What is the most purchased item on the menu and how many times was it purchased by all customers?
Which item was the most popular for each customer?
Which item was purchased first by the customer after they became a member?
Which item was purchased just before the customer became a member?
What is the total items and amount spent for each member before they became a member?
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
