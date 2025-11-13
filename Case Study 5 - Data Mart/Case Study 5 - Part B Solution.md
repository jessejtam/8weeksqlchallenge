### 1. What day of the week is used for each week_date value?
````sql
SELECT
	DISTINCT(TO_CHAR(week_date, 'Day')) AS day
FROM clean_weekly_sales;
````

### 2. What range of week numbers are missing from the dataset?
````sql
SELECT
	DISTINCT(week_number)
FROM clean_weekly_sales
ORDER BY week_number ASC;
````

### 3. How many total transactions were there for each year in the dataset?
````sql
SELECT
	DATE_PART('Year', week_date) AS year,
    SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY year
ORDER BY year ASC;
````

### 4. What is the total sales for each region for each month?
````sql
SELECT
	region,
	TO_CHAR(week_date, 'Month') AS month,
    SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY region, month
ORDER BY region, month;
````

### 5. What is the total count of transactions for each platform?
````sql
SELECT
	platform,
    SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform;
````

### 6. What is the percentage of sales for Retail vs Shopify for each month?
````sql
WITH retail AS (
	SELECT
  		TO_CHAR(week_date, 'FMMonth') AS month,
  		SUM(sales) AS retail_sales
  	FROM clean_weekly_sales
  	WHERE platform = 'Retail'
  	GROUP BY month
),

shopify AS (
	SELECT
  		TO_CHAR(week_date, 'FMMonth') AS month,
  		SUM(sales) AS shopify_sales
  	FROM clean_weekly_sales
  	WHERE platform = 'Shopify'
  	GROUP BY month
)

SELECT
	month,
    ROUND(100 * retail_sales :: DECIMAL / (retail_sales + shopify_sales), 2) AS retail_percent,
    ROUND(100 * shopify_sales :: DECIMAL / (retail_sales + shopify_sales), 2) AS shopify_percent
FROM retail
LEFT JOIN shopify
USING(month)
ORDER BY
	CASE WHEN month = 'March' THEN 1
    	WHEN month = 'April' THEN 2
        WHEN month = 'May' THEN 3
        WHEN month = 'June' THEN 4
        WHEN month = 'July' THEN 5
        WHEN month = 'August' THEN 6
        WHEN month = 'September' THEN 7
    END;
````

### 7. What is the percentage of sales by demographic for each year in the dataset?
````sql
WITH total AS (
	SELECT
  		DATE_PART('Year', week_date) AS year,
  		SUM(sales) AS total_sales
  	FROM clean_weekly_sales
  	GROUP BY year
),

demographics AS (
	SELECT
  		DATE_PART('Year', week_date) AS year,
  		demographic,
  		SUM(sales) AS demographic_sales
  	FROM clean_weekly_sales
  	GROUP BY year, demographic
)

SELECT
	year,
    demographic,
    ROUND(100 * demographic_sales :: DECIMAL / total_sales, 2) AS percent_of_sales
FROM total
LEFT JOIN demographics
USING(year)
ORDER BY year;
````

### 8. Which age_band and demographic values contribute the most to Retail sales?
````sql
SELECT
	age_band,
    demographic,
    SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE platform = 'Retail' AND age_band <> 'unknown'
GROUP BY age_band, demographic
ORDER BY total_sales DESC;
````

### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
````sql
SELECT
	calendar_year,
    platform,
    ROUND(AVG(avg_transaction), 0) AS method1,
    SUM(sales) / SUM(transactions) AS method2
FROM clean_weekly_sales
GROUP BY calendar_year, platform
ORDER BY calendar_year, platform;
````
