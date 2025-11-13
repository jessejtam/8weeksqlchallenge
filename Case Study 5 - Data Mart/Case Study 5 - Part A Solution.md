````sql
CREATE TABLE clean_weekly_sales AS
	SELECT
    	TO_DATE(week_date, 'DD-MM-YY') AS week_date,
        DATE_PART('week', TO_DATE(week_date, 'DD-MM-YY')) AS week_number,
        DATE_PART('month', TO_DATE(week_date,'DD-MM-YY')) AS month_number,
        DATE_PART('year', TO_DATE(week_date,'DD-MM-YY')) AS calendar_year,
        CASE WHEN segment = 'null' THEN 'unknown'
        ELSE segment END AS segment,
        CASE WHEN segment LIKE '_1' THEN 'Young Adults'
        	WHEN segment LIKE '_2' THEN 'Middle Aged'
            WHEN segment LIKE '_3' OR segment LIKE '_4' THEN 'Retirees'
        ELSE 'unkwown' END AS age_band,
        CASE WHEN segment LIKE 'C_' THEN 'Couples'
        	WHEN segment LIKE 'F_' THEN 'Families'
        ELSE 'unknown' END AS demographic,
        ROUND(sales :: DECIMAL / transactions, 2) AS avg_transaction
    FROM data_mart.weekly_sales;
````
