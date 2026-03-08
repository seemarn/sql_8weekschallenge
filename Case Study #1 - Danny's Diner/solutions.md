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

```sql
WITH popularity AS(
SELECT
  	customer_id,
    product_name,
    COUNT(product_name) AS count,
    DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS rnk
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id, count DESC)

SELECT customer_id, product_name, count
FROM popularity
WHERE rnk = 1
```

### Answer
| customer_id | product_name | count |
| ----------- | ------------ | ----- |
| A           | ramen        | 3     |
| B           | ramen        | 2     |
| B           | curry        | 2     |
| B           | sushi        | 2     |
| C           | ramen        | 3     |


**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH members_first_purchased AS (
SELECT 
	s.customer_id,
    s.order_date,
    m.product_name,
    DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date) AS rnk
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mem ON s.customer_id = mem.customer_id
WHERE join_date < order_date
ORDER BY s.customer_id, s.order_date)

SELECT customer_id, product_name
FROM members_first_purchased
WHERE rnk = 1
```

### Answer
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |


**7. Which item was purchased just before the customer became a member?**

```sql
WITH members_purchased_before AS(
SELECT 
	s.customer_id,
    s.order_date,
    m.product_name,
    RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) AS rnk
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mem ON s.customer_id = mem.customer_id
WHERE join_date > order_date
ORDER BY s.customer_id, s.order_date)

SELECT customer_id, product_name
FROM members_purchased_before
WHERE rnk = 1
```

### Answer
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT 
	s.customer_id,
    COUNT(s.product_id) AS count_items,
    SUM(price) AS amount_spent
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mem ON s.customer_id = mem.customer_id
WHERE join_date > order_date
GROUP BY s.customer_id
ORDER BY s.customer_id
```

### Answer
| customer_id | count_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |


**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
WITH menu_points AS(
SELECT 
    product_id,
    CASE WHEN product_id = 1 THEN price * 20 ELSE price * 10 END AS points
FROM menu)

SELECT 
	customer_id,
    SUM(points) AS total_points
FROM sales s
LEFT JOIN menu_points mp ON s.product_id = mp.product_id
GROUP BY customer_id
ORDER BY customer_id
```

### Answer
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
WITH menu_points AS(
SELECT 
  product_id,
  (price*20) AS first_week_points,
  CASE WHEN product_id = 1 THEN price * 20 ELSE price * 10 END AS points
FROM menu)

SELECT 
	s.customer_id,
    SUM(CASE WHEN (s.order_date-mem.join_date) BETWEEN 0 AND 6 THEN first_week_points ELSE points END) AS total_points
FROM sales s
LEFT JOIN members mem ON s.customer_id = mem.customer_id
LEFT JOIN menu_points mp ON s.product_id = mp.product_id
WHERE s.order_date >= mem.join_date AND s.order_date BETWEEN '2021-01-01' AND '2021-01-31' 
GROUP BY s.customer_id
ORDER BY s.customer_id
```

### Answer
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1020         |
| B           | 320          |

## Bonus Questions

### Join All The Things

```sql
SELECT
  	s.customer_id,
    order_date,
    product_name,
    price,
    CASE WHEN mem.join_date <= s.order_date then 'Y' ELSE 'N' END AS member
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mem ON s.customer_id = mem.customer_id
ORDER BY s.customer_id, order_date, product_name
```

### Answer
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

### Rank All The Things

```sql
WITH member_check AS (
    SELECT
        s.customer_id,
        s.order_date,
        m.product_name,
        m.price,
        CASE WHEN mem.join_date <= s.order_date THEN 'Y' ELSE 'N' END AS member
    FROM sales s
    LEFT JOIN menu m ON s.product_id = m.product_id
    LEFT JOIN members mem ON s.customer_id = mem.customer_id
)

SELECT *,
    CASE WHEN member = 'Y' 
    THEN DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date) 
        ELSE NULL 
    END AS ranking
FROM member_check
ORDER BY customer_id, order_date, product_name
```

### Answer
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |
