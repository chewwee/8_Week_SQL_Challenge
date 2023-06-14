## Case Study #4 Data Bank
<img src="https://8weeksqlchallenge.com/images/case-study-designs/4.png" style="height:500px;width:500px" />

## Entity Relationship Diagram
<img src="https://github.com/chewwee/8_Week_SQL_Challenge/blob/6094e64aa8d8c4c9a9045a65cc362752db18110d/Case%20Study%20%234%20-%20Data%20Bank/Data%20Bank.png" style="height:300px;width:500px"/>

## Case Study Questions
## A. Customer Nodes Exploration

#### 1. How many unique nodes are there on the Data Bank system?
```SQL
SELECT 
  COUNT(
    DISTINCT(node_id)
  ) AS num_of_nodes 
FROM 
  data_bank.customer_nodes;
```
|num_of_nodes|
|------------|
| 5          |
 
#### 2. What is the number of nodes per region?
```SQL
SELECT 
  region_name, 
  COUNT(
    DISTINCT(node_id)
  ) AS num_of_nodes 
FROM 
  data_bank.customer_nodes c 
  JOIN data_bank.regions r ON c.region_id = r.region_id 
GROUP BY 
  region_name 
ORDER BY 
  region_name;

```
| region_name | num_of_nodes |
| ----------- | ------------ |
|  Africa     | 5            |
|  America    | 5            |
|  Asia       | 5            |
| Australia   | 5            |
|  Europe     | 5            |
 
#### 3.  How many customers are allocated to each region?
```SQL
SELECT 
  region_name, 
  COUNT(
    DISTINCT(customer_id)
  ) AS num_of_cust 
FROM 
  data_bank.customer_nodes c
JOIN 
  data_bank.regions r
ON
  c.region_id = r.region_id
GROUP BY 
  region_name 
ORDER BY 
  region_name;
```
| region_name | num_of_cust |
| ----------- | ------------ |
|  Africa     |    102       |
|  America    |    105       |
|  Asia       |    95        |
| Australia   |    110       |
|  Europe     |    88        |

#### 4. How many days on average are customers reallocated to a different node?
```SQL
WITH node_days AS (
  SELECT 
    customer_id, 
    node_id, 
    end_date - start_date as date_diff 
  FROM 
    data_bank.customer_nodes 
  WHERE 
    end_date != '9999-12-31' 
  GROUP BY 
    customer_id, 
    node_id, 
    start_date, 
    end_date
), 
sum_node_days AS (
  SELECT
    customer_id, 
    node_id, 
    SUM(date_diff) as sum_of_node_days 
  FROM
    node_days 
  GROUP BY 
    customer_id, 
    node_id 
  ORDER BY 
    customer_id
) 
SELECT
  ROUND(AVG(sum_of_node_days)) AS avg_node_rellocation_days
FROM
  sum_node_days
```

| avg_node_rellocation_days|
| ------------------------- |
| 24                         


#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```SQL
WITH node_days AS (
  SELECT 
    c.customer_id, 
    r.region_name, 
    c.node_id, 
    c.start_date, 
    c.end_date, 
    c.end_date - c.start_date AS date_diff 
  FROM 
    data_bank.customer_nodes c 
    JOIN data_bank.regions r ON c.region_id = r.region_id 
  WHERE 
    c.end_date != '9999-12-31'
) 
SELECT 
  region_name, 
  PERCENTILE_DISC(0.5) WITHIN GROUP (
    ORDER BY 
      date_diff
  ) AS median, 
  PERCENTILE_DISC(0.8) WITHIN GROUP (
    ORDER BY 
      date_diff
  ) AS percentile_80, 
  PERCENTILE_DISC(0.95) WITHIN GROUP (
    ORDER BY 
      date_diff
  ) AS percentile_95 
FROM 
  node_days 
GROUP BY 
  region_name
```

| region_name | median | percentile_80 | percentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 15     | 24            | 28            |
| America     | 15     | 23            | 28            |
| Asia        | 15     | 23            | 28            |
| Australia   | 15     | 23            | 28            |
| Europe      | 15     | 24            | 28            |







## B. Customer Transactions
#### 1. What is the unique count and total amount for each transaction type?
```SQL
SELECT 
  txn_type, 
  COUNT(txn_type) AS Trans_Count, 
  SUM(txn_Amount) AS Trans_Amt 
FROM 
  data_bank.customer_transactions 
GROUP BY 
  txn_type;
```
| txn_type   | trans_ count | trans_amt | 
| ---------- | ------------ | --------- |
| purchase   | 1617         | 806537    |
| deposit    | 2671         | 1359168   |
| withdrawal | 1580         | 793003    |

#### 2. What is the average total historical deposit counts and amounts for all customers?
```SQL
WITH cte AS (
  SELECT 
    customer_id, 
    COUNT(*) AS trans_count, 
    AVG(txn_amount) AS trans_amount 
  FROM 
    data_bank.customer_transactions 
  WHERE 
    txn_type = 'deposit' 
  GROUP BY 
    customer_id 
  order by 
    customer_id
) 
SELECT 
  ROUND(
    AVG(trans_count)
  ) AS avg_count, 
  ROUND(
    AVG(trans_amount)
  ) AS avg_amount 
FROM 
  cte
```
| avg_count | avg_amount | 
| --------- | ---------- |
| 5         | 509        |

#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```SQL
WITH CTE AS (
  SELECT 
    DATE_PART('MONTH', txn_date) AS mth, 
    customer_id, 
    SUM(
      CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END
    ) AS deposit_count, 
    SUM(
      CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END
    ) AS purchase_count, 
    SUM(
      CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END
    ) AS withdrawal_count 
  FROM 
    data_bank.customer_transactions 
  GROUP BY 
    mth, 
    customer_id 
  ORDER BY 
    mth
) 
SELECT 
  mth, 
  COUNT(
    DISTINCT(customer_id)
  ) AS num_of_cust 
FROM 
  CTE 
WHERE 
  deposit_count > 1 
  AND (
    purchase_count = 1 
    OR withdrawal_count = 1
  ) 
GROUP BY 
  mth;
```
| mth | num_of_cust | 
| --- | ----------- |
| 1   | 115         |
| 2   | 108         |
| 3   | 113         |
| 4   | 50          |


#### 4. What is the closing balance for each customer at the end of the month?
#### 5. What is the percentage of customers who increase their closing balance by more than 5%?

