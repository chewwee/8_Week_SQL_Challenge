## Case Study #1 - Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" style="height: 500px; width:500px;/>

#### 1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT 
  s.customer_id, 
  SUM(m.price) AS total_amount 
FROM 
  dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id;
```
Answer:
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

#### 2. How many days has each customer visited the restaurant?

```SQL
SELECT 
  customer_id, 
  COUNT(
    DISTINCT(order_date)
  ) AS num_of_days 
FROM 
  dannys_diner.sales 
GROUP BY 
  customer_id;
```
Answer:
| customer_id | num_of_days |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

#### 3. What was the first item from the menu purchased by each customer?

```SQL
WITH CTE AS (
  SELECT 
    ROW_NUMBER() OVER (PARTITION BY customer_id) AS row_num, 
    customer_id, 
    order_date, 
    product_id 
  FROM 
    dannys_diner.sales
) 
SELECT 
  CTE.customer_id, 
  m.product_name 
FROM 
  CTE 
  JOIN dannys_diner.menu m ON CTE.product_id = m.product_id 
WHERE 
  row_num = 1;
```
Answer:
| customer_id | product_name |
| -----------  | ------------ |
| A              | sushi        |
| B           | curry        |
| C           | ramen        |

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT 
  m.product_name, 
  count(s.product_id) AS num_of_purchased 
FROM 
  dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
GROUP BY 
  s.product_id, 
  m.product_name 
ORDER BY 
  num_of_purchased DESC
LIMIT 
  1;
```
Answer:
| product_name | num_of_purchased |
| ------------ | ---------------- |
| ramen        | 8                |


#### 5. Which item was the most popular for each customer?

```SQL
WITH CTE AS (
  SELECT 
    DENSE_RANK () OVER (
      PARTITION BY customer_id 
      ORDER BY 
        COUNT(product_id) desc
    ) AS row_num, 
    customer_id, 
    product_id, 
    COUNT(product_id) AS num_of_purchased 
  FROM 
    dannys_diner.sales 
  GROUP BY 
    customer_id, 
    product_id
) 
SELECT 
  CTE.customer_id, 
  m.product_name, 
  CTE.num_of_purchased 
FROM 
  CTE 
  JOIN dannys_diner.menu m ON CTE.product_id = m.product_id 
WHERE 
  CTE.row_num = 1 
ORDER BY 
  CTE.customer_id;
```

Answer:
| customer_id | product_name | num_of_purchased |
| ----------- | ------------ | ---------------- |
| A           | ramen        | 3                |
| B           | sushi        | 2                |
| B           | curry        | 2                |
| B           | ramen        | 2                |
| C           | ramen        | 3                |

#### 6. Which item was purchased first by the customer after they became a member?

```SQL
WITH CTE AS (
  SELECT 
    ROW_NUMBER() OVER (PARTITION BY s.customer_id) AS row_num, 
    s.customer_id, 
    s.product_id 
  FROM 
    dannys_diner.sales s 
    RIGHT JOIN dannys_diner.members mb ON s.customer_id = mb.customer_id 
  WHERE 
    mb.join_date < s.order_date
) 
SELECT 
  CTE.customer_id, 
  m.product_name 
FROM 
  CTE 
  JOIN dannys_diner.menu m ON CTE.product_id = m.product_id 
WHERE 
  CTE.row_num = 1 
ORDER BY 
  CTE.customer_id;
```
Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

#### 7. Which item was purchased just before the customer became a member?

```SQL
WITH CTE AS (
  SELECT 
    ROW_NUMBER() OVER (
      PARTITION BY s.customer_id 
      ORDER BY 
        s.order_date DESC
    ) AS row_num, 
    s.customer_id, 
    s.product_id, 
    s.order_date, 
    mb.join_date 
  FROM 
    dannys_diner.sales s 
    RIGHT JOIN dannys_diner.members mb ON s.customer_id = mb.customer_id 
  WHERE 
    S.order_date < mb.join_date
) 
SELECT 
  CTE.customer_id, 
  m.product_name 
FROM 
  CTE 
  JOIN dannys_diner.menu m ON CTE.product_id = m.product_id 
WHERE 
  CTE.row_num = 1 
ORDER BY 
  CTE.customer_id;
```
Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

#### 8.What is the total items and amount spent for each member before they became a member?

```SQL
SELECT 
  s.customer_id, 
  COUNT(s.product_id) AS num_of_items_purchased, 
  SUM(m.price) as total_amount 
FROM 
  dannys_diner.sales s 
  RIGHT JOIN dannys_diner.members mb ON s.customer_id = mb.customer_id 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
WHERE 
  s.order_date < mb.join_date 
GROUP BY 
  s.customer_id 
ORDER BY 
  s.customer_id;
```
Answer:
| customer_id | num_of_items_purchased | total_amount |
| ----------- | ---------------------- | ------------ |
| A           | 2                      | 25           |
| B           | 3                      | 40           |

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```SQL
SELECT 
  s.customer_id, 
  SUM(
    CASE WHEN product_name = 'sushi' THEN price * 20 ELSE price * 10 END
  ) AS point 
FROM 
  dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
GROUP BY 
  s.customer_id 
ORDER BY 
  customer_id;
```
Answer:
| customer_id | point |
| ----------- | ----- |
| A           | 860   |
| B           | 940   |
| C           | 360   |

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```SQL
SELECT 
  s.customer_id, 
  SUM(
    CASE WHEN s.order_date >= mb.join_date 
    AND s.order_date <= mb.join_date + 6 THEN m.price * 20 WHEN m.product_name = 'sushi' THEN m.price * 20 ELSE m.price * 10 END
  ) AS points 
FROM 
  dannys_diner.sales s 
  RIGHT JOIN dannys_diner.members mb ON s.customer_id = mb.customer_id 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
WHERE 
  EXTRACT(
    MONTH 
    FROM 
      s.order_date
  ) = 1 
GROUP BY 
  s.customer_id 
ORDER BY 
  s.customer_id;
```
Answer:
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |





