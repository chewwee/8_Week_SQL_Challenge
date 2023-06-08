## A. Pizza Metrics

1. How many pizzas were ordered?
```SQL
SELECT 
  COUNT(*) 
FROM 
  pizza_runner.customer_orders;
```
Answer:
| count |
| ----- |
| 14    |

2. How many unique customer orders were made?
```SQL
    SELECT 
      COUNT(
        DISTINCT(order_id)
      ) 
    FROM 
      pizza_runner.customer_orders;
```

| count |
| ----- |
| 10    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

3. How many successful orders were delivered by each runner?
```SQL
SELECT 
  runner_id, 
  COUNT(runner_id) 
FROM 
  pizza_runner.runner_orders 
WHERE 
  pickup_time NOT LIKE 'null' 
GROUP BY 
  runner_id 
ORDER BY 
  runner_id;
```

Answer:
| runner_id | count |
| --------- | ----- |
| 1         | 4     |
| 2         | 3     |
| 3         | 1     |


4. How many of each type of pizza was delivered?
```SQL
SELECT 
  pn.pizza_name, 
  COUNT(cust.pizza_id) 
FROM 
  pizza_runner.customer_orders cust 
  JOIN pizza_runner.runner_orders runner ON cust.order_id = runner.order_id 
  JOIN pizza_runner.pizza_names pn ON cust.pizza_id = pn.pizza_id 
WHERE 
  runner.pickup_time NOT LIKE 'null' 
GROUP BY 
  pn.pizza_name;
```

| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |


5. How many Vegetarian and Meatlovers were ordered by each customer?
```SQL
WITH cte AS (
  SELECT 
    cust.customer_id, 
    pn.pizza_name 
  FROM 
    pizza_runner.customer_orders cust 
    JOIN pizza_runner.pizza_names pn ON cust.pizza_id = pn.pizza_id
) 
SELECT 
  customer_id, 
  SUM(
    (pizza_name = 'Meatlovers'):: INT
  ) Meatlovers, 
  SUM(
    (pizza_name = 'Vegetarian'):: INT
  ) Vegetarian 
FROM 
  cte 
GROUP BY 
  customer_id 
ORDER BY 
  customer_id;
```
```SQL
WITH cte AS (
  SELECT 
    cust.customer_id, 
    pn.pizza_name 
  FROM 
    pizza_runner.customer_orders cust 
    JOIN pizza_runner.pizza_names pn ON cust.pizza_id = pn.pizza_id
) 
SELECT 
  customer_id, 
  COUNT(*) FILTER (
    WHERE 
      pizza_name = 'Meatlovers'
  ) Meatlovers, 
  COUNT(*) FILTER (
    WHERE 
      pizza_name = 'Vegetarian'
  ) Vegetarian 
FROM 
  cte 
GROUP BY 
  customer_id 
ORDER BY 
  customer_id;
```

Answer:
| customer_id | meatlovers | vegetarian |
| ----------- | ---------- | ---------- |
| 101         | 2          | 1          |
| 102         | 2          | 1          |
| 103         | 3          | 1          |
| 104         | 3          | 0          |
| 105         | 0          | 1          |



6. What was the maximum number of pizzas delivered in a single order?
```SQL
SELECT 
  COUNT(cust.pizza_id) AS num_of_pizza 
FROM 
  pizza_runner.customer_orders cust 
  JOIN pizza_runner.runner_orders runner ON cust.order_id = runner.order_id 
WHERE 
  runner.pickup_time NOT LIKE 'null' 
GROUP BY 
  cust.order_id 
ORDER BY 
  num_of_pizza DESC 
LIMIT 
  1;
```

| num_of_pizza |
| ------------ |
| 3            |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```SQL
WITH delivered_pizza AS (
  SELECT 
    * 
  FROM 
    pizza_runner.customer_orders cust 
    JOIN pizza_runner.runner_orders runner ON cust.order_id = runner.order_id 
  WHERE 
    runner.pickup_time NOT LIKE 'null'
), 
CTE AS (
  SELECT 
    customer_id, 
    count(*) as times, 
    'changes' as status 
  FROM 
    delivered_pizza 
  WHERE 
    exclusions ~ '[0-9]' 
    OR extras ~ '[0-9]' 
  GROUP BY 
    customer_id 
  UNION 
  SELECT 
    customer_id, 
    count(*) as times, 
    'no changes' as status 
  FROM 
    delivered_pizza 
  WHERE 
    exclusions ! ~ '[0-9]' 
    AND extras ! ~ '[0-9]' 
    OR extras IS Null 
  GROUP BY 
    customer_id
) 
SELECT 
  customer_id, 
  SUM(times) FILTER (
    WHERE 
      status = 'changes'
  ) as changes, 
  SUM(times) FILTER (
    WHERE 
      status = 'no changes'
  ) as no_changes 
FROM 
  CTE 
GROUP BY 
  customer_id 
ORDER BY 
  customer_id;
```
Answer:

| customer_id | changes | no_changes |
| ----------- | ------- | ---------- |
| 101         |         | 2          |
| 102         |         | 3          |
| 103         | 3       |            |
| 104         | 2       | 1          |
| 105         | 1       |            |


8. How many pizzas were delivered that had both exclusions and extras？

```SQL
WITH delivered_pizza AS (
  SELECT 
    * 
  FROM 
    pizza_runner.customer_orders cust 
    JOIN pizza_runner.runner_orders runner ON cust.order_id = runner.order_id 
  WHERE 
    runner.pickup_time NOT LIKE 'null'
), 
CTE AS (
  SELECT 
    customer_id, 
    count(*) as times, 
    'changes' as status 
  FROM 
    delivered_pizza 
  WHERE 
    exclusions ~ '[0-9]' 
    AND extras ~ '[0-9]' 
  GROUP BY 
    customer_id
) 
SELECT 
  SUM(times) 
FROM 
  CTE;
```

| sum |
| --- |
| 1   |



9. What was the total volume of pizzas ordered for each hour of the day?
```SQL
SELECT 
  EXTRACT(
    HOUR 
    FROM 
      order_time
  ) as order_hour, 
  count(*) 
FROM 
  pizza_runner.customer_orders 
GROUP BY 
  order_hour 
ORDER BY 
  order_hour;
```
Answer:

| order_hour | count |
| ---------- | ----- |
| 11         | 1     |
| 13         | 3     |
| 18         | 3     |
| 19         | 1     |
| 21         | 3     |
| 23         | 3     |


10. What was the volume of orders for each day of the week?
```SQL
SELECT 
  EXTRACT(
    DOW 
    FROM 
      order_time
  ) as order_day, 
  count(*) 
FROM 
  pizza_runner.customer_orders 
GROUP BY 
  order_day 
ORDER BY 
  order_day;
```
Answer:
| order_day | count |
| --------- | ----- |
| 3         | 5     |
| 4         | 3     |
| 5         | 1     |
| 6         | 5     |
