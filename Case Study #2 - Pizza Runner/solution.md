Case Study #2 - Pizza Runner

https://8weeksqlchallenge.com/case-study-2/

- [Pizza Metrics](#a-pizza-metrics)
- [Runner and Customer Experience](### B. Runner and Customer Experience)

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
SELECT co.customer_id, co.order_id,
COUNT(co.pizza_id) AS num
FROM customer_orders co
LEFT JOIN runner_orders pn ON co.order_id = pn.order_id
WHERE distance != 0
GROUP BY co.order_id, co.customer_id
ORDER BY num DESC
LIMIT 1
```

**Answer**

<img width="309" height="93" alt="image" src="https://github.com/user-attachments/assets/4e652f0e-391a-4c8a-bc6f-80ba29942638" />


**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
SELECT 
	co.customer_id, 
    SUM(CASE 
        WHEN (co.exclusions IS NULL OR co.exclusions IN ('', 'null')) 
         AND (co.extras IS NULL OR co.extras IN ('', 'null')) 
        THEN 1 ELSE 0 END) AS no_changes,
    SUM(CASE 
        WHEN (co.exclusions IS NOT NULL AND co.exclusions NOT IN ('', 'null')) 
          OR (co.extras IS NOT NULL AND co.extras NOT IN ('', 'null')) 
        THEN 1 ELSE 0 END) AS with_changes
FROM customer_orders co
JOIN runner_orders pn ON co.order_id = pn.order_id
WHERE pn.distance != '0'
GROUP BY co.customer_id;
```

**Answer**

<img width="403" height="214" alt="image" src="https://github.com/user-attachments/assets/723dc34e-740e-4415-b5d9-39d6566fdb85" />


**8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT
	co.customer_id, 
    SUM(CASE 
        WHEN (co.exclusions IS NOT NULL AND co.exclusions NOT IN ('', 'null')) 
          AND (co.extras IS NOT NULL AND co.extras NOT IN ('', 'null')) 
        THEN 1 ELSE 0 END) AS with_changes
FROM customer_orders co
JOIN runner_orders pn ON co.order_id = pn.order_id
WHERE pn.distance != 0
GROUP BY co.customer_id
```

**Answer**

<img width="282" height="199" alt="image" src="https://github.com/user-attachments/assets/421b4bd4-4abf-4661-a130-323f42393241" />


**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT
  EXTRACT(HOUR FROM order_time) AS hours,
  COUNT(order_id) AS num_pizza
FROM customer_orders
GROUP BY hours
ORDER BY hours
```

**Answer**

<img width="215" height="230" alt="image" src="https://github.com/user-attachments/assets/d6c25e73-325b-48b9-b17c-ae738ecd33e1" />


**10. What was the volume of orders for each day of the week?**

```sql
SELECT 
    DAYNAME(order_time) AS day_of_week,
    COUNT(order_id) AS num_orders
FROM customer_orders
GROUP BY day_of_week, DAYOFWEEK(order_time)
ORDER BY DAYOFWEEK(order_time);
```

**Answer**

<img width="282" height="184" alt="image" src="https://github.com/user-attachments/assets/1a8d35db-f08d-4ebc-b562-5ae5df3f8f09" />

### B. Runner and Customer Experience
