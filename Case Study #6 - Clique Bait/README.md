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
#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

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
