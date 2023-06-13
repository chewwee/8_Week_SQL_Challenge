## Case Study #5 - Data Mart

<img src="https://8weeksqlchallenge.com/images/case-study-designs/5.png" style="height:500px; width:500px"/>

## Entity Relationship Diagram

## Case Study Questions
## 1. Data Cleansing Steps

## 2. Data Exploration
#### 1. What day of the week is used for each week_date value?
#### 2. What range of week numbers are missing from the dataset?
#### 3. How many total transactions were there for each year in the dataset?
#### 4. What is the total sales for each region for each month?
#### 5. What is the total count of transactions for each platform
#### 6. What is the percentage of sales for Retail vs Shopify for each month?
#### 7. What is the percentage of sales by demographic for each year in the dataset?
#### 8. Which age_band and demographic values contribute the most to Retail sales?
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
