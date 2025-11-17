### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
Start by getting the Week Number of 2020-06-15:
````sql
SELECT DATE_PART('week', '2020-06-15' :: DATE);
````

````sql
WITH before_change AS (
	SELECT
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 21 AND 24 AND calendar_year = 2020
),

after_change AS (
	SELECT
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 28 AND calendar_year = 2020
)

SELECT
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
CROSS JOIN after_change;
````

### 2. What about the entire 12 weeks before and after?
````sql
WITH before_change AS (
	SELECT
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
),

after_change AS (
	SELECT
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
)

SELECT
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
CROSS JOIN after_change;
````

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
For 4 weeks:
````sql
WITH before_change AS (
	SELECT
  		calendar_year,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 21 AND 24
  	GROUP BY calendar_year
),

after_change AS (
	SELECT
  		calendar_year,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 28
  	GROUP BY calendar_year
)

SELECT
	calendar_year,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(calendar_year)
ORDER BY calendar_year;
````

For 12 weeks:
````sql
WITH before_change AS (
	SELECT
  		calendar_year,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24
  	GROUP BY calendar_year
),

after_change AS (
	SELECT
  		calendar_year,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36
  	GROUP BY calendar_year
)

SELECT
	calendar_year,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(calendar_year)
ORDER BY calendar_year;
````
