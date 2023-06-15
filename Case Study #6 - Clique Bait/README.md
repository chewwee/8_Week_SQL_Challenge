## Case Study #6 - Clique Bait 
<img src="https://8weeksqlchallenge.com/images/case-study-designs/6.png" style="height:500px; width:500px"/>

## Entity Relationship Diagram

## Case Study Questions

## 2. Digital Analysis
#### 1. How many users are there?
```sql
SELECT 
  COUNT(
    DISTINCT(user_id)
  ) AS user_count 
FROM 
  clique_bait.users;
```
| user_count |
| ----- |
| 500   |

#### 2. How many cookies does each user have on average?
```sql
SELECT 
  ROUND(
    COUNT(cookie_id):: numeric / COUNT(
      DISTINCT(user_id)
    )
  ) AS avg_cookie 
FROM 
  clique_bait.users;
```
| avg_cookie |
| ---------- |
| 4          |

#### 3. What is the unique number of visits by all users per month?
```sql
SELECT 
  date_part('month', event_time) as month_number, 
  count(
    distinct(visit_id)
  ) AS num_of_visit 
FROM 
  clique_bait.events 
GROUP BY 
  month_number;
```
| month_number | num_of_visit |
| ------------ | ---------- |
| 1            | 876        |
| 2            | 1488       |
| 3            | 916        |
| 4            | 248        |
| 5            | 36         |

#### 4. What is the number of events for each event type?
```sql
SELECT 
  ei.event_name, 
  count(e.event_type) as event_num 
FROM 
  clique_bait.events e 
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type 
GROUP BY 
  ei.event_name;
```
| event_name    | event_num |
| ------------- | --------- |
| Purchase      | 1777      |
| Ad Impression | 876       |
| Add to Cart   | 8451      |
| Page View     | 20928     |
| Ad Click      | 702       |

#### 5. What is the percentage of visits which have a purchase event?
```sql
SELECT 
  ROUND(
    SUM(
      CASE WHEN event_type = '3' THEN 1 ELSE 0 END
    ) * 100 :: numeric / COUNT(
      DISTINCT(visit_id)
    ), 
    2
  ) AS percent_of_purchase 
FROM 
  clique_bait.events;
```
| percent_of_purchase |
| ------------------- |
| 49.86               |


#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
SELECT 
  100 - ROUND(
    (
      SUM(
        CASE WHEN e.event_type = '3' THEN 1 ELSE 0 END
      )* 100 :: numeric / SUM(
        CASE WHEN e.event_type = '1' 
        AND ph.page_id = '12' THEN 1 ELSE 0 END
      )
    ), 
    2
  ) AS percentage_of_not_purchase 
FROM 
  clique_bait.events e 
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type 
  JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
```
| percentage_of_not_purchase |
| -------------------------- |
| 15.50                      |

#### 7. What are the top 3 pages by number of views?
```sql
SELECT 
  ph.page_name, 
  COUNT(e.event_type) AS count_of_view 
FROM 
  clique_bait.events e 
  JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id 
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type 
WHERE 
  ei.event_name = 'Page View' 
GROUP BY 
  ph.page_name 
ORDER BY 
  count_of_view desc 
LIMIT 
  3;
```
| page_name    | count_of_view |
| ------------ | ------------- |
| All Products | 3174          |
| Checkout     | 2103          |
| Home Page    | 1782          |

#### 8. What is the number of views and cart adds for each product category?
```sql
SELECT 
  ph.product_category, 
  SUM(
    CASE WHEN ei.event_name = 'Page View' THEN 1 ELSE 0 END
  ) AS count_of_views, 
  SUM(
    CASE WHEN ei.event_name = 'Add to Cart' THEN 1 ELSE 0 END
  ) AS count_of_addCart 
FROM 
  clique_bait.events e 
  JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id 
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
WHERE 
  ph.product_category IS NOT NULL
GROUP BY 
  ph.product_category
```
| product_category | count_of_views | count_of_addcart |
| ---------------- | -------------- | ---------------- |
| Luxury           | 3032           | 1870             |
| Shellfish        | 6204           | 3792             |
| Fish             | 4633           | 2789             |

#### 9. What are the top 3 products by purchases?

## 3. Product Funnel Analysis
#### 1. Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

```sql
WITH view_cart as (
  SELECT 
    e.visit_id, 
    ph.page_name, 
    ph.product_category, 
    SUM(
      CASE WHEN e.event_type = '1' THEN 1 ELSE 0 END
    ) AS page_view, 
    SUM(
      CASE WHEN e.event_type = '2' THEN 1 ELSE 0 END
    ) AS add_to_cart 
  FROM 
    clique_bait.events e 
    JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id 
  WHERE 
    ph.product_id IS NOT NULL 
  GROUP BY 
    e.visit_id, 
    ph.page_name, 
    ph.product_category 
  ORDER BY 
    e.visit_id
), 
purchase as (
  SELECT 
    distinct(visit_id) 
  FROM 
    clique_bait.events 
  WHERE 
    event_type = '3'
), 
view_cart_purchase as (
  SELECT 
    vc.visit_id, 
    vc.page_name, 
    vc.product_category, 
    vc.page_view, 
    vc.add_to_cart, 
    CASE WHEN P.visit_id IS NOT NULL THEN 1 ELSE 0 END AS purchase 
  FROM 
    view_cart vc 
    LEFT JOIN purchase p ON vc.visit_id = p.visit_id
), 
full_table as(
  SELECT 
    visit_id, 
    page_name, 
    product_category, 
    page_view, 
    add_to_cart, 
    purchase, 
    CASE WHEN add_to_cart = 1 
    AND purchase = 0 THEN 1 ELSE 0 END AS abandoned 
  FROM 
    view_cart_purchase
) 
select 
  page_name, 
  product_category, 
  sum(page_view) as view, 
  sum(add_to_cart) as cart, 
  sum(purchase) as purchase, 
  sum(abandoned) as abandoned 
from 
  full_table 
group by 
  page_name, 
  product_category;
```
| page_name      | product_category | view | cart | purchase | abandoned |
| -------------- | ---------------- | ---- | ---- | -------- | --------- |
| Salmon         | Fish             | 1559 | 938  | 1094     | 227       |
| Oyster         | Shellfish        | 1568 | 943  | 1136     | 217       |
| Tuna           | Fish             | 1515 | 931  | 1058     | 234       |
| Russian Caviar | Luxury           | 1563 | 946  | 1088     | 249       |
| Kingfish       | Fish             | 1559 | 920  | 1106     | 213       |
| Black Truffle  | Luxury           | 1469 | 924  | 1045     | 217       |
| Crab           | Shellfish        | 1564 | 949  | 1108     | 230       |
| Lobster        | Shellfish        | 1547 | 968  | 1113     | 214       |
| Abalone        | Shellfish        | 1525 | 932  | 1085     | 233       |


Use your 2 new output tables - answer the following questions:

#### 1. Which product had the most views, cart adds and purchases?
#### 2. Which product was most likely to be abandoned?
#### 3. Which product had the highest view to purchase percentage?
#### 4. What is the average conversion rate from view to cart add?
#### 5. What is the average conversion rate from cart add to purchase?

## 4. Campaigns Analysis
Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.
