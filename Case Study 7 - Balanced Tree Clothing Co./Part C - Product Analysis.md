# Part C - Product Analysis
## 1. What are the top 3 products by total revenue before discount?
#### SQL Query
````sql
SELECT
	product_name,
    SUM(qty * s.price) AS total_revenue
FROM balanced_tree.sales AS s
LEFT JOIN balanced_tree.product_details AS d
ON s.prod_id = d.product_id
GROUP BY product_name
ORDER BY total_revenue DESC
LIMIT 3;
````
#### Final Output

## 2. What is the total quantity, revenue and discount for each segment?
#### SQL Query
````sql
SELECT
	segment_id,
    segment_name,
    SUM(qty) AS total_quantity,
    ROUND(SUM(qty * s.price * (100 - discount) / 100.0), 2) AS total_revenue,
    ROUND(SUM(qty * s.price * discount / 100.0), 2) AS total_discount
FROM balanced_tree.sales AS s
LEFT JOIN balanced_tree.product_details AS d
ON s.prod_id = d.product_id
GROUP BY segment_id, segment_name
ORDER BY segment_id ASC;
````
#### Final Output

## 3. What is the top selling product for each segment?
#### SQL Query
````sql
WITH top_product AS (
	SELECT
    	segment_name,
    	product_name,
    	SUM(qty) AS total_quantity,
  		RANK() OVER(PARTITION BY segment_name ORDER BY SUM(qty) DESC) AS quantity_rank
	FROM balanced_tree.sales AS s
	LEFT JOIN balanced_tree.product_details AS d
	ON s.prod_id = d.product_id
	GROUP BY segment_name, product_name
)

SELECT
	segment_name,
    product_name,
    total_quantity
FROM top_product
WHERE quantity_rank = 1;
````
#### Final Output

## 4. What is the total quantity, revenue and discount for each category?
#### SQL Query
````sql
SELECT
	category_id,
    category_name,
    SUM(qty) AS total_quantity,
    ROUND(SUM(qty * s.price * (100 - discount) / 100.0), 2) AS total_revenue,
    ROUND(SUM(qty * s.price * discount / 100.0), 2) AS total_discount
FROM balanced_tree.sales AS s
LEFT JOIN balanced_tree.product_details AS d
ON s.prod_id = d.product_id
GROUP BY category_id, category_name
ORDER BY category_id ASC;
````
#### Final Output

## 5. What is the top selling product for each category?
#### SQL Query
````sql
WITH top_product AS (
	SELECT
    	category_name,
    	product_name,
    	SUM(qty) AS total_quantity,
  		RANK() OVER(PARTITION BY category_name ORDER BY SUM(qty) DESC) AS quantity_rank
	FROM balanced_tree.sales AS s
	LEFT JOIN balanced_tree.product_details AS d
	ON s.prod_id = d.product_id
	GROUP BY category_name, product_name
)

SELECT
	category_name,
    product_name,
    total_quantity
FROM top_product
WHERE quantity_rank = 1;
````
#### Final Output

## 6. What is the percentage split of revenue by product for each segment?
#### SQL Query
````sql
WITH product AS (
	SELECT
  		segment_id,
  		segment_name,
  		product_name,
  		SUM(qty * s.price * (100 - discount) / 100.0) AS product_revenue
  	FROM balanced_tree.sales AS s
  	LEFT JOIN balanced_tree.product_details AS d
  	ON s.prod_id = d.product_id
  	GROUP BY 1, 2, 3
)

SELECT
	segment_name,
    product_name,
	ROUND(100 *product_revenue / SUM(product_revenue) OVER(PARTITION BY segment_name), 2) AS percent_split
FROM product
ORDER BY segment_id;
````
#### Final Output

## 7. What is the percentage split of revenue by segment for each category?
#### SQL Query
````sql
WITH segment AS (
	SELECT
  		category_id,
  		category_name,
  		segment_name,
  		SUM(qty * s.price * (100 - discount) / 100.0) AS segment_revenue
  	FROM balanced_tree.sales AS s
  	LEFT JOIN balanced_tree.product_details AS d
  	ON s.prod_id = d.product_id
  	GROUP BY 1, 2, 3
)

SELECT
	category_name,
    segment_name,
	ROUND(100 * segment_revenue / SUM(segment_revenue) OVER(PARTITION BY category_name), 2) AS percent_split
FROM segment
ORDER BY category_id;
````
#### Final Output

## 8. What is the percentage split of total revenue by category?
#### SQL Query
````sql
SELECT
    ROUND(100 * SUM(CASE WHEN category_id = 1 THEN qty * s.price * (100 - discount) / 100.0 ELSE NULL END) / SUM(qty * s.price * (100 - discount) / 100.0), 2) AS women_percent,
    ROUND( 100 * SUM(CASE WHEN category_id = 2 THEN qty * s.price * (100 - discount) / 100.0 ELSE NULL END) / SUM(qty * s.price * (100 - discount) / 100.0), 2) AS men_percent 
FROM balanced_tree.sales AS s
LEFT JOIN balanced_tree.product_details AS d
ON s.prod_id = d.product_id;
````
#### Final Output

## 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
#### SQL Query
````sql
SELECT
	product_name,
    ROUND(100.0 * COUNT(*) / (SELECT COUNT(DISTINCT(txn_id)) FROM balanced_tree.sales), 2) AS penetration
FROM balanced_tree.sales AS s
LEFT JOIN balanced_tree.product_details AS d
ON s.prod_id = d.product_id
GROUP BY product_name;
````
#### Final Output

## 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
#### SQL Query
````sql
WITH combinations AS (
	SELECT
  		ARRAY[t1.product_name, t2.product_name, t3.product_name] AS product_array,
		t1.product_name || ', ' || t2.product_name || ', ' || t3.product_name AS product_combination
	FROM balanced_tree.product_details AS t1
	JOIN balanced_tree.product_details AS t2
	ON t1.style_id < t2.style_id
	JOIN balanced_tree.product_details AS t3
	ON t2.style_id < t3.style_id
),

transactions AS (
	SELECT
  		txn_id,
  		ARRAY_AGG(product_name ORDER BY style_id ASC) AS transaction_combination
  	FROM balanced_tree.product_details AS d
  	JOIN balanced_tree.sales AS s
  	ON d.product_id = s.prod_id
  	GROUP BY txn_id
)

SELECT
	product_combination,
    COUNT(*) AS times_combined
FROM combinations AS c
JOIN transactions AS t
ON t.transaction_combination @> c.product_array
GROUP BY product_combination
ORDER BY times_combined DESC
LIMIT 1;
````
#### Final Output
