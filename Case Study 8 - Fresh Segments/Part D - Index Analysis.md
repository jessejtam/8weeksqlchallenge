# Part D - Index Analysis
## 1. What is the top 10 interests by the average composition for each month?
#### SQL Query
````sql
WITH ranked_table AS (
	SELECT
  		month_year,
  		id,
  		interest_name,
  		ROUND((composition / index_value) :: DECIMAL, 2) AS avg_composition,
  		RANK() OVER(PARTITION BY month_year ORDER BY composition / index_value DESC) AS composition_rank
  	FROM fresh_segments.interest_metrics AS t1
  	JOIN fresh_segments.interest_map AS t2
  	ON t1.interest_id :: INTEGER = t2.id
)

SELECT *
FROM ranked_table
WHERE composition_rank <= 10
ORDER BY month_year ASC, composition_rank ASC;
````
#### Final Output 

## 2. For all of these top 10 interests - which interest appears the most often?
#### SQL Query
````sql
WITH ranked_table AS (
	SELECT
  		month_year,
  		id,
  		interest_name,
  		ROUND((composition / index_value) :: DECIMAL, 2) AS avg_composition,
  		RANK() OVER(PARTITION BY month_year ORDER BY composition / index_value DESC) AS composition_rank
  	FROM fresh_segments.interest_metrics AS t1
  	JOIN fresh_segments.interest_map AS t2
  	ON t1.interest_id :: INTEGER = t2.id
)

SELECT
	id,
    interest_name,
    COUNT(*) AS times_appeared
FROM ranked_table
WHERE composition_rank <= 10
GROUP BY 1, 2
ORDER BY 3 DESC;
````
#### Final Output 

## 3. What is the average of the average composition for the top 10 interests for each month?
#### SQL Query
````sql
WITH ranked_table AS (
	SELECT
  		month_year,
  		id,
  		interest_name,
  		ROUND((composition / index_value) :: DECIMAL, 2) AS avg_composition,
  		RANK() OVER(PARTITION BY month_year ORDER BY composition / index_value DESC) AS composition_rank
  	FROM fresh_segments.interest_metrics AS t1
  	JOIN fresh_segments.interest_map AS t2
  	ON t1.interest_id :: INTEGER = t2.id
)

SELECT
	month_year,
    ROUND(AVG(avg_composition), 2) AS avg_avg_composition
FROM ranked_table
WHERE composition_rank <= 10
GROUP BY 1
ORDER BY 1;
````
#### Final Output 

## 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
#### SQL Query
````sql
WITH ranked_table AS (
	SELECT
  		month_year,
  		interest_name,
  		ROUND((composition / index_value) :: DECIMAL, 2) AS max_index_composition,
  		RANK() OVER(PARTITION BY month_year ORDER BY composition / index_value DESC) AS composition_rank
  	FROM fresh_segments.interest_metrics AS t1
  	JOIN fresh_segments.interest_map AS t2
  	ON t1.interest_id :: INTEGER = t2.id
),

rank_one AS(
	SELECT
		month_year,
    	interest_name,
    	max_index_composition,
    	ROUND(AVG(max_index_composition) OVER(ORDER BY month_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS "3_month_moving_avg",
    	CONCAT(LAG(interest_name) OVER(ORDER BY month_year) || ': ' || LAG(max_index_composition) OVER(ORDER BY month_year)) AS "1_month_ago",
    	CONCAT(LAG(interest_name, 2) OVER(ORDER BY month_year) || ': ' || LAG(max_index_composition, 2) OVER(ORDER BY month_year)) AS "2_months_ago"
	FROM ranked_table
	WHERE composition_rank = 1
)

SELECT *
FROM rank_one
WHERE month_year >= '2018-09-01';
````
#### Final Output 
