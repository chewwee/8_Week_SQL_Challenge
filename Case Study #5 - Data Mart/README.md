## Case Study #5 - Data Mart

<img src="https://8weeksqlchallenge.com/images/case-study-designs/5.png" style="height:500px; width:500px"/>

## Entity Relationship Diagram

## Case Study Questions
## 1. Data Cleansing Steps

In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

1. Convert the week_date to a DATE format
2. Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
3. Add a month_number with the calendar month for each week_date value as the 3rd column
4. Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
5. Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
6. Add a new demographic column using the following mapping for the first letter in the segment values:
7. Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
8. Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

```SQL
DROP TABLE IF EXISTS clean_weekly_sales;
CREATE TEMPORARY TABLE clean_weekly_sales AS (
  SELECT 
    TO_DATE(week_date, 'dd/mm/yy') AS week_date, 
    DATE_PART(
      'week', 
      TO_DATE(week_date, 'dd/mm/yy')
    ) AS week_number, 
    DATE_PART(
      'month', 
      TO_DATE(week_date, 'dd/mm/yy')
    ) AS month_number, 
    DATE_PART(
      'year', 
      TO_DATE(week_date, 'dd/mm/yy')
    ) AS calender_number, 
    region, 
    platform, 
    CASE WHEN segment = 'null' THEN 'unknown' ELSE segment END AS segment, 
    CASE WHEN RIGHT(segment, 1)= '1' THEN 'Young Adults' WHEN RIGHT(segment, 1)= '2' THEN 'Middle Aged' WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees' ELSE 'unknown' END AS age_band, 
    CASE WHEN LEFT(segment, 1)= 'C' THEN 'Couples' WHEN left(segment, 1)= 'F' THEN 'Families' ELSE 'unknown' END AS demographic, 
    customer_type, 
    transactions, 
    sales, 
    ROUND(sales :: numeric / transactions, 2) AS avg_transaction 
  FROM 
    data_mart.weekly_sales
```
| week_date                | week_number | month_number | calender_number | region | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | --------------- | ------ | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | ASIA   | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.31           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | ASIA   | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.56           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | USA    | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 16509610 | 31.20           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | EUROPE | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.42           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | AFRICA | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.29           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | CANADA | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.54          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | AFRICA | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.64          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | ASIA   | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.11          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | AFRICA | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.84          |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020            | AFRICA | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 3888162  | 35.02           |



## 2. Data Exploration
#### 1. What day of the week is used for each week_date value?
```SQL
SELECT 
  DISTINCT(
    TO_CHAR(week_date, 'day')
  ) AS DAY 
FROM 
  clean_weekly_sales;
```
| DAY    |
| ------ |
| monday |

#### 2. What range of week numbers are missing from the dataset?
#### 3. How many total transactions were there for each year in the dataset?
```SQL
SELECT 
  calender_number, 
  SUM(transactions) AS total_transactions 
FROM 
  clean_weekly_sales 
GROUP BY 
  calender_number 
ORDER BY 
  calender_number;
```
| calender_number | total_transactions |
| --------------- | ------------------ |
| 2018            | 346406460          |
| 2019            | 365639285          |
| 2020            | 375813651          |

#### 4. What is the total sales for each region for each month?
```SQL
SELECT 
  region, 
  month_number, 
  SUM(sales) AS total_sales 
FROM 
  clean_weekly_sales 
GROUP BY 
  region, 
  month_number 
ORDER BY 
  region, 
  month_number;
```
| region        | month_number | total_sales |
| ------------- | ------------ | ----------- |
| AFRICA        | 3            | 567767480   |
| AFRICA        | 4            | 1911783504  |
| AFRICA        | 5            | 1647244738  |
| AFRICA        | 6            | 1767559760  |
| AFRICA        | 7            | 1960219710  |
| AFRICA        | 8            | 1809596890  |
| AFRICA        | 9            | 276320987   |
| ASIA          | 3            | 529770793   |
| ASIA          | 4            | 1804628707  |
| ASIA          | 5            | 1526285399  |
| ASIA          | 6            | 1619482889  |
| ASIA          | 7            | 1768844756  |
| ASIA          | 8            | 1663320609  |
| ASIA          | 9            | 252836807   |
| CANADA        | 3            | 144634329   |
| CANADA        | 4            | 484552594   |
| CANADA        | 5            | 412378365   |
| CANADA        | 6            | 443846698   |
| CANADA        | 7            | 477134947   |
| CANADA        | 8            | 447073019   |
| CANADA        | 9            | 69067959    |
| EUROPE        | 3            | 35337093    |
| EUROPE        | 4            | 127334255   |
| EUROPE        | 5            | 109338389   |
| EUROPE        | 6            | 122813826   |
| EUROPE        | 7            | 136757466   |
| EUROPE        | 8            | 122102995   |
| EUROPE        | 9            | 18877433    |
| OCEANIA       | 3            | 783282888   |
| OCEANIA       | 4            | 2599767620  |
| OCEANIA       | 5            | 2215657304  |
| OCEANIA       | 6            | 2371884744  |
| OCEANIA       | 7            | 2563459400  |
| OCEANIA       | 8            | 2432313652  |
| OCEANIA       | 9            | 372465518   |
| SOUTH AMERICA | 3            | 71023109    |
| SOUTH AMERICA | 4            | 238451531   |
| SOUTH AMERICA | 5            | 201391809   |
| SOUTH AMERICA | 6            | 218247455   |
| SOUTH AMERICA | 7            | 235582776   |
| SOUTH AMERICA | 8            | 221166052   |
| SOUTH AMERICA | 9            | 34175583    |
| USA           | 3            | 225353043   |
| USA           | 4            | 759786323   |
| USA           | 5            | 655967121   |
| USA           | 6            | 703878990   |
| USA           | 7            | 760331754   |
| USA           | 8            | 712002790   |
| USA           | 9            | 110532368   |

#### 5. What is the total count of transactions for each platform
```sql
SELECT 
  platform, 
  SUM(transactions) AS total_transactions 
FROM 
  clean_weekly_sales 
GROUP BY 
  platform;
```
| platform | total_transactions |
| -------- | ------------------ |
| Shopify  | 5925169            |
| Retail   | 1081934227         |

#### 6. What is the percentage of sales for Retail vs Shopify for each month?
```sql
SELECT 
  DISTINCT calender_number, 
  month_number, 
  platform, 
  round(
    (
      (
        SUM(sales) OVER (
          PARTITION BY calender_number, month_number, 
          platform
        )
      ) * 100
    ):: numeric / SUM(sales) OVER (
      PARTITION BY calender_number, month_number
    ), 
    2
  ) as percentage_of_sales 
FROM 
  clean_weekly_sales 
ORDER BY 
  calender_number, 
  month_number;
```
| calender_number | month_number | platform | percentage_of_sales |
| --------------- | ------------ | -------- | ------------------- |
| 2018            | 3            | Shopify  | 2.08                |
| 2018            | 3            | Retail   | 97.92               |
| 2018            | 4            | Shopify  | 2.07                |
| 2018            | 4            | Retail   | 97.93               |
| 2018            | 5            | Shopify  | 2.27                |
| 2018            | 5            | Retail   | 97.73               |
| 2018            | 6            | Retail   | 97.76               |
| 2018            | 6            | Shopify  | 2.24                |
| 2018            | 7            | Retail   | 97.75               |
| 2018            | 7            | Shopify  | 2.25                |
| 2018            | 8            | Shopify  | 2.29                |
| 2018            | 8            | Retail   | 97.71               |
| 2018            | 9            | Shopify  | 2.32                |
| 2018            | 9            | Retail   | 97.68               |
| 2019            | 3            | Shopify  | 2.29                |
| 2019            | 3            | Retail   | 97.71               |
| 2019            | 4            | Shopify  | 2.20                |
| 2019            | 4            | Retail   | 97.80               |
| 2019            | 5            | Shopify  | 2.48                |
| 2019            | 5            | Retail   | 97.52               |
| 2019            | 6            | Shopify  | 2.58                |
| 2019            | 6            | Retail   | 97.42               |
| 2019            | 7            | Shopify  | 2.65                |
| 2019            | 7            | Retail   | 97.35               |
| 2019            | 8            | Shopify  | 2.79                |
| 2019            | 8            | Retail   | 97.21               |
| 2019            | 9            | Shopify  | 2.91                |
| 2019            | 9            | Retail   | 97.09               |
| 2020            | 3            | Retail   | 97.30               |
| 2020            | 3            | Shopify  | 2.70                |
| 2020            | 4            | Retail   | 96.96               |
| 2020            | 4            | Shopify  | 3.04                |
| 2020            | 5            | Shopify  | 3.29                |
| 2020            | 5            | Retail   | 96.71               |
| 2020            | 6            | Shopify  | 3.20                |
| 2020            | 6            | Retail   | 96.80               |
| 2020            | 7            | Retail   | 96.67               |
| 2020            | 7            | Shopify  | 3.33                |
| 2020            | 8            | Retail   | 96.51               |
| 2020            | 8            | Shopify  | 3.49                |

#### 7. What is the percentage of sales by demographic for each year in the dataset?
```sql
SELECT 
  DISTINCT calender_number, 
  demographic, 
  ROUND(
    (
      SUM(sales) OVER (
        PARTITION BY calender_number, demographic
      )* 100 :: numeric / SUM(sales) OVER (PARTITION BY calender_number)
    ), 
    2
  ) as demographic_sales 
FROM 
  clean_weekly_sales 
ORDER BY 
  calender_number;
```
| calender_number | demographic | demo_sales |
| --------------- | ----------- | ---------- |
| 2018            | unknown     | 41.63      |
| 2018            | Families    | 31.99      |
| 2018            | Couples     | 26.38      |
| 2019            | Couples     | 27.28      |
| 2019            | unknown     | 40.25      |
| 2019            | Families    | 32.47      |
| 2020            | Couples     | 28.72      |
| 2020            | unknown     | 38.55      |
| 2020            | Families    | 32.73      |

#### 8. Which age_band and demographic values contribute the most to Retail sales?
```sql
SELECT 
  age_band, 
  demographic, 
  SUM(sales) AS retail_sales 
FROM 
  clean_weekly_sales 
WHERE 
  platform = 'Retail' 
GROUP BY 
  age_band, 
  demographic 
ORDER BY 
  retail_sales desc;
```
| age_band     | demographic | retail_sales |
| ------------ | ----------- | ------------ |
| unknown      | unknown     | 16067285533  |
| Retirees     | Families    | 6634686916   |
| Retirees     | Couples     | 6370580014   |
| Middle Aged  | Families    | 4354091554   |
| Young Adults | Couples     | 2602922797   |
| Middle Aged  | Couples     | 1854160330   |
| Young Adults | Families    | 1770889293   |

#### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
## 3. Before & After Analysis
This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.
Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.
We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before
Using this analysis approach - answer the following questions:
1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
2. What about the entire 12 weeks before and after?
3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
## 4. Bonus Question
Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
- region
- platform
- age_band
- demographic
- customer_type
Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?
