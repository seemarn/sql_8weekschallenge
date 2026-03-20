Case Study #2 - Pizza Runner

https://8weeksqlchallenge.com/case-study-2/

- [Pizza Metrics](#a-pizza-metrics)
- [Runner and Customer Experience](#b-runner-and-customer-experience)

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
**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```sql
SELECT 
    FLOOR(DATEDIFF(registration_date, '2021-01-01') / 7) + 1 AS reg_week,
    COUNT(runner_id) AS num_signup
FROM runners
GROUP BY reg_week
```

**Answer**

<img width="255" height="141" alt="image" src="https://github.com/user-attachments/assets/ae322d08-b9b4-4257-9335-447e22f1aea3" />


**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
WITH time_to_reach AS (
SELECT 
    runner_id,
    CAST(ro.pickup_time AS TIME) AS pickup_time,
    CAST(co.order_time AS TIME) AS order_time,
    TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS time_taken
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.distance != 0
ORDER BY runner_id)

SELECT 
    runner_id, 
    ROUND(AVG(time_taken), 2) AS avg_time_taken
FROM time_to_reach
GROUP BY runner_id
```

**Answer**

<img width="292" height="142" alt="image" src="https://github.com/user-attachments/assets/5ed7d4c6-1cd3-44b9-ae59-9410b739e51b" />


**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
WITH time_table AS (
SELECT 
    ro.order_id,
    COUNT(pizza_id) AS num_pizza,
    CAST(ro.pickup_time AS TIME) AS pickup_time,
    CAST(co.order_time AS TIME) AS order_time,
    TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS time_taken
FROM runner_orders ro
LEFT JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.distance != 0
GROUP BY ro.order_id, pickup_time, order_time)

SELECT 
    num_pizza,
    ROUND(AVG(time_taken), 2) AS avg_time_per_order,
    ROUND(AVG(time_taken) / num_pizza, 2) AS avg_time_per_pizza
FROM time_table
GROUP BY num_pizza
ORDER BY num_pizza
```

**Answer**

<img width="487" height="148" alt="image" src="https://github.com/user-attachments/assets/9c1adeb6-4e8c-4554-9162-46c9587dc743" />


**4. What was the average distance travelled for each customer?**

```sql
SELECT 
    customer_id,
    ROUND(AVG(distance),2) AS avg_dist
FROM customer_orders co
JOIN runner_orders ro ON co.order_id = ro.order_id
WHERE distance != 0
GROUP BY customer_id
ORDER BY customer_id
```

**Answer**

<img width="233" height="191" alt="image" src="https://github.com/user-attachments/assets/f3a44ee4-4cb3-457a-befa-86d3acacd92f" />


**5. What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT 
    MAX(duration) AS max_duration,
    CAST(MIN(duration) AS FLOAT) AS min_duration,
    MAX(duration) - MIN(duration) AS diff
FROM runner_orders ro 
WHERE distance != 0
```

**Answer**

<img width="353" height="98" alt="image" src="https://github.com/user-attachments/assets/489610bf-9db3-4cef-8177-a854ac82a0e8" />


**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
speed = distance/hour

```sql

```

**Answer**


**7. What is the successful delivery percentage for each runner?**

```sql

```

**Answer**

