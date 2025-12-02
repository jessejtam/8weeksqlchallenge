# Part B - Transaction Analysis
## 1. How many unique transactions were there?
#### SQL Query
````sql
SELECT
	COUNT(DISTINCT(txn_id)) AS unique_transactions
FROM balanced_tree.sales;
````
#### Final Output

## 2. What is the average unique products purchased in each transaction?
#### SQL Query
````sql
WITH products_per_transaction AS (
	SELECT
  		txn_id,
  		COUNT(DISTINCT(prod_id)) AS unique_products
  	FROM balanced_tree.sales
  	GROUP BY txn_id
)

SELECT
	ROUND(AVG(unique_products), 3) AS avg_unique_products
FROM products_per_transaction;
````
#### Final Output

## 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
#### SQL Query
````sql
WITH revenue_per_transaction AS(
	SELECT
  		txn_id,
  		SUM(qty * price * (100 - discount) / 100.0) AS revenue
  	FROM balanced_tree.sales
  	GROUP BY txn_id
)

SELECT
	ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) :: DECIMAL, 2) AS twenty_fifth_percentile,
    ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY revenue) :: DECIMAL, 2) AS fiftieth_percentile,
    ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) :: DECIMAL, 2) AS seventy_fifth_percentile
FROM revenue_per_transaction;
````
#### Final Output

## 4. What is the average discount value per transaction?
#### SQL Query
````sql
WITH discount_per_transaction AS (
	SELECT
  		txn_id,
  		SUM(qty * price * discount / 100.0) AS discount_amount
  	FROM balanced_tree.sales
  	GROUP BY txn_id
)

SELECT
	ROUND(AVG(discount_amount), 2) AS avg_discount
FROM discount_per_transaction;
````
#### Final Output

## 5. What is the percentage split of all transactions for members vs non-members?
#### SQL Query
````sql
WITH members AS (
	SELECT
  		txn_id,
		member
  	FROM balanced_tree.sales
  	GROUP BY txn_id, member
)

SELECT
	ROUND(100.0 * SUM(CASE WHEN member = 't' THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_members,
    ROUND(100.0 * SUM(CASE WHEN member = 'f' THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_non_members
FROM members;
````
#### Final Output

## 6. What is the average revenue for member transactions and non-member transactions?
#### SQL Query
````sql
WITH members AS (
	SELECT
  		txn_id,
  		SUM(CASE WHEN member = 't' THEN qty * price * (100 - discount) / 100.0 ELSE NULL END) AS member_revenue,
        SUM(CASE WHEN member = 'f' THEN qty * price * (100 - discount) / 100.0 ELSE NULL END) AS non_member_revenue
  	FROM balanced_tree.sales
  	GROUP BY txn_id
)

SELECT
	ROUND(AVG(member_revenue), 2) AS avg_member_revenue,
    ROUND(AVG(non_member_revenue), 2) AS avg_non_member_revenue
FROM members;
````
#### Final Output
