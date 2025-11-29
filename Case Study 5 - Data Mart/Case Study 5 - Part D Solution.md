### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
- region
- platform
- age_band
- demographic
- customer_type
## Region Analysis
#### SQL Query
````sql
WITH before_change AS (
	SELECT
  		region,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
  	GROUP BY region
),

after_change AS (
	SELECT
  		region,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
  	GROUP BY region
)

SELECT
	region,
	sales_before,
  sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(region)
ORDER BY percent_difference;
````
#### Final Output
<img width="753" height="215" alt="image" src="https://github.com/user-attachments/assets/d75e649c-a171-4cf2-8d69-776c5909cc06" />

## Platform Analysis
#### SQL Query
````sql
WITH before_change AS (
	SELECT
  		platform,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
  	GROUP BY platform
),

after_change AS (
	SELECT
  		platform,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
  	GROUP BY platform
)

SELECT
	platform,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(platform)
ORDER BY percent_difference;
````
#### Final Output
<img width="747" height="100" alt="image" src="https://github.com/user-attachments/assets/8e598bcc-5282-434d-b341-d33a720a536d" />

## Age Band Analysis
#### SQL Query
````sql
WITH before_change AS (
	SELECT
  		age_band,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
  	GROUP BY age_band
),

after_change AS (
	SELECT
  		age_band,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
  	GROUP BY age_band
)

SELECT
	age_band,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(age_band)
ORDER BY percent_difference;
````
#### Final Output
<img width="758" height="143" alt="image" src="https://github.com/user-attachments/assets/085e47c7-0499-4169-9254-1b2796e01024" />

## Demographic Analysis
#### SQL Query
````sql
WITH before_change AS (
	SELECT
  		demographic,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
  	GROUP BY demographic
),

after_change AS (
	SELECT
  		demographic,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
  	GROUP BY demographic
)

SELECT
	demographic,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(demographic)
ORDER BY percent_difference;
````
#### Final Output
<img width="746" height="121" alt="image" src="https://github.com/user-attachments/assets/20ee2c63-9581-49b3-a5da-42645192f724" />

## Customer Type Analysis
#### SQL Query
````sql
WITH before_change AS (
	SELECT
  		customer_type,
  		SUM(sales) AS sales_before
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 13 AND 24 AND calendar_year = 2020
  	GROUP BY customer_type
),

after_change AS (
	SELECT
  		customer_type,
  		SUM(sales) AS sales_after
  	FROM clean_weekly_sales
  	WHERE week_number BETWEEN 25 AND 36 AND calendar_year = 2020
  	GROUP BY customer_type
)

SELECT
	customer_type,
	sales_before,
    sales_after,
    sales_after - sales_before AS total_sale_difference,
    ROUND(100 * sales_after :: DECIMAL / sales_before - 100, 2) AS percent_difference
FROM before_change
LEFT JOIN after_change
USING(customer_type)
ORDER BY percent_difference;
````
#### Final Output
<img width="766" height="126" alt="image" src="https://github.com/user-attachments/assets/b417ebfd-579e-4a47-9f45-ebaf2deba4f5" />
