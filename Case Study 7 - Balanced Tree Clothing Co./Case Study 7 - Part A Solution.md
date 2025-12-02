# Part A - High Level Sales Analysis
## 1. What was the total quantity sold for all products?
#### SQL Query
````sql
SELECT
	SUM(qty) AS total_sold
FROM balanced_tree.sales;
````
#### Final Output

## 2. What is the total generated revenue for all products before discounts?
#### SQL Query
````sql
SELECT
	SUM(qty * price) AS generated_revenue
FROM balanced_tree.sales;
````
#### Final Output

## 3. What was the total discount amount for all products?
#### SQL Query
````sql
SELECT
	ROUND(SUM(qty * price * discount / 100.0), 2) AS total_discount
FROM balanced_tree.sales;
````
#### Final Output
