# Part B - Data Exploration
## 1. What day of the week is used for each week_date value?
#### SQL Query
````sql
SELECT
	DISTINCT(TO_CHAR(week_date, 'Day')) AS day
FROM clean_weekly_sales;
````
#### Final Output
<img width="88" height="85" alt="image" src="https://github.com/user-attachments/assets/cfd4154f-55e5-42d0-88a8-7c09adcf8b12" />

## 2. What range of week numbers are missing from the dataset?
#### SQL Query
````sql
WITH weeks AS (
    SELECT generate_series(1, 52) AS missing_weeks
)

SELECT
	missing_weeks
FROM weeks
WHERE missing_weeks NOT IN (SELECT DISTINCT(week_number) FROM clean_weekly_sales)
ORDER BY missing_weeks ASC;
````
#### Final Output
<img width="98" height="801" alt="image" src="https://github.com/user-attachments/assets/36075390-aa57-455d-8a31-2638512f0ba5" />

## 3. How many total transactions were there for each year in the dataset?
#### SQL Query
````sql
SELECT
	DATE_PART('Year', week_date) AS year,
    SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY year
ORDER BY year ASC;
````
#### Final Output
<img width="272" height="138" alt="image" src="https://github.com/user-attachments/assets/9ba6f0f9-7ba6-4874-a109-1cd7f594e442" />

## 4. What is the total sales for each region for each month?
#### SQL Query
````sql
SELECT
	region,
	DATE_PART('Month', week_date) AS month,
    SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY region, month
ORDER BY region, month;
````
#### Final Output
Only showing results for Canada because the actual output is too long.
<img width="383" height="245" alt="image" src="https://github.com/user-attachments/assets/c3b9df90-640b-47d4-afc9-ecf0589a4be2" />

## 5. What is the total count of transactions for each platform?
#### SQL Query
````sql
SELECT
	platform,
    SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform;
````
#### Final Output
<img width="270" height="97" alt="image" src="https://github.com/user-attachments/assets/3636c257-13dc-489f-bfc6-16ad4e2c9634" />

## 6. What is the percentage of sales for Retail vs Shopify for each month?
#### SQL Query
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
#### Final Output
<img width="415" height="218" alt="image" src="https://github.com/user-attachments/assets/907ce307-8e0b-460d-a55f-17ef40e23258" />

## 7. What is the percentage of sales by demographic for each year in the dataset?
#### SQL Query
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
#### Final Output

## 8. Which age_band and demographic values contribute the most to Retail sales?
#### SQL Query
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
#### Final Output

## 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
#### SQL Query
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
#### Final Output
