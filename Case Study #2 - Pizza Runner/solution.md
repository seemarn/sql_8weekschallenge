Case Study #2 - Pizza Runner

https://8weeksqlchallenge.com/case-study-2/

## Entity Relationship Diagram
<img width="598" height="307" alt="image" src="https://github.com/user-attachments/assets/de68cd21-5980-4496-a864-c3a1c70f28b4" />


## Questions and Solutions
### A. Pizza Metrics
**1. How many pizzas were ordered?**

```sql
SELECT COUNT(*) AS count_pizzas_order
FROM customer_orders
```

**Answer**
<img width="224" height="100" alt="image" src="https://github.com/user-attachments/assets/b6d14240-3adc-4b02-b4e9-4099c56063fc" />


**2. How many unique customer orders were made?**

```sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders
```

**Answer**
<img width="263" height="108" alt="image" src="https://github.com/user-attachments/assets/71c02b5e-a950-42d9-83a5-1b3bb6ea6c1b" />


**3. How many successful orders were delivered by each runner?**

```sql
SELECT runner_id, 
COUNT(DISTINCT order_id) AS num_success_orders
FROM runner_orders
WHERE distance != 0
GROUP BY runner_id
```

**Answer**
<img width="322" height="147" alt="image" src="https://github.com/user-attachments/assets/3399f0c9-e3ff-48ea-abfa-ab42874c0c1c" />


**4. How many of each type of pizza was delivered?**

```sql
SELECT co.pizza_id, pn.pizza_name,
COUNT(co.pizza_id) AS num_delivered
FROM customer_orders co
LEFT JOIN runner_orders ro ON co.order_id = ro.order_id 
LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
WHERE ro.distance != 0
GROUP BY co.pizza_id, pn.pizza_name
```

**Answer**
<img width="378" height="138" alt="image" src="https://github.com/user-attachments/assets/37b05eda-f5fc-4aa9-bfde-e0a90163b1fd" />


**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT co.customer_id, pn.pizza_name,
COUNT(co.pizza_id) AS num
FROM customer_orders co
LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
GROUP BY co.customer_id, pn.pizza_name
ORDER BY co.customer_id, pn.pizza_name
```

**Answer**
<img width="339" height="284" alt="image" src="https://github.com/user-attachments/assets/244d9455-50b6-4137-9e3e-310524613af8" />


**6. What was the maximum number of pizzas delivered in a single order?**

```sql

```

**Answer**


**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql

```

**Answer**


**8. How many pizzas were delivered that had both exclusions and extras?**

```sql

```

**Answer**


**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql

```

**Answer**


**10. What was the volume of orders for each day of the week?**

```sql

```

**Answer**

### B. Runner and Customer Experience
