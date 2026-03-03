# Case Study #1 - Danny's Diner

https://8weeksqlchallenge.com/case-study-1/

## Questions and Solutions
**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT 
  s.customer_id AS customer_id,
  SUM(price) AS total_amount,
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id; 
```

### Answer
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

**2. How many days has each customer visited the restaurant?**

```sql
SELECT 
	customer_id,
    COUNT(DISTINCT order_date) AS total_days_visit
FROM sales
GROUP BY customer_id
ORDER BY customer_id;
```

### Answer
| customer_id | total_days_visit |
| ----------- | ---------------- |
| A           | 4                |
| B           | 6                |
| C           | 2                |


**3. What was the first item from the menu purchased by each customer?**

```sql
SELECT DISTINCT
  s.customer_id AS customer_id,
  m.product_name AS first_item_purchased
FROM sales s
JOIN menu m ON s.product_id = m.product_id
WHERE (s.customer_id, s.order_date) IN(
  SELECT customer_id, MIN(order_date) AS first_date
  FROM sales
  GROUP BY customer_id)
ORDER BY s.customer_id;
```

### Answer
| customer_id | first_item_purchased |
| ----------- | -------------------- |
| A           | curry                |
| A           | sushi                |
| B           | curry                |
| C           | ramen                |

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT 
    m.product_name AS product_name,
    COUNT(s.product_id) AS frequency
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY frequency DESC
LIMIT 1;
```

### Answer
| product_name | frequency |
| ------------ | --------- |
| ramen        | 8         |


**5. Which item was the most popular for each customer?**

### Answer

**6. Which item was purchased first by the customer after they became a member?**

### Answer

**7. Which item was purchased just before the customer became a member?**

### Answer

**8. What is the total items and amount spent for each member before they became a member?**

### Answer

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

### Answer

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

### Answer
