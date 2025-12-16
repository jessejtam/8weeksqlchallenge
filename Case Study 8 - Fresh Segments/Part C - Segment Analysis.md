# Part C - Segment Analysis
## 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
Top 10:
#### SQL Query
````sql
WITH filtered AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
  	HAVING COUNT(DISTINCT(month_year)) >= 6
),

ranked_table AS (
	SELECT
		id,
    	interest_name,
    	month_year,
    	composition,
    	RANK() OVER(PARTITION BY id ORDER BY composition DESC) AS composition_rank
	FROM fresh_segments.interest_metrics AS t1
	JOIN fresh_segments.interest_map AS t2
	ON t1.interest_id :: INTEGER = t2.id
	WHERE id IN (SELECT interest_id :: INTEGER FROM filtered)
)

SELECT
	id,
    interest_name,
    month_year,
    composition
FROM ranked_table
WHERE composition_rank = 1
ORDER BY composition DESC
LIMIT 10;
````
#### Final Output

Bottom 10: 
#### SQL Query
````sql
WITH filtered AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
  	HAVING COUNT(DISTINCT(month_year)) >= 6
),

ranked_table AS (
	SELECT
		id,
    	interest_name,
    	month_year,
    	composition,
    	RANK() OVER(PARTITION BY id ORDER BY composition DESC) AS composition_rank
	FROM fresh_segments.interest_metrics AS t1
	JOIN fresh_segments.interest_map AS t2
	ON t1.interest_id :: INTEGER = t2.id
	WHERE id IN (SELECT interest_id :: INTEGER FROM filtered)
)

SELECT
	id,
    interest_name,
    month_year,
    composition
FROM ranked_table
WHERE composition_rank = 1
ORDER BY composition ASC
LIMIT 10;
````
#### Final Output

## 2. Which 5 interests had the lowest average ranking value?
#### SQL Query
````sql
WITH filtered AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
  	HAVING COUNT(DISTINCT(month_year)) >= 6
)

SELECT
	id,
    interest_name,
    ROUND(AVG(ranking), 2) AS avg_ranking
FROM fresh_segments.interest_metrics AS t1
JOIN fresh_segments.interest_map AS t2
ON t1.interest_id :: INTEGER = t2.id
WHERE id IN (SELECT interest_id :: INTEGER FROM filtered)
GROUP BY 1, 2
ORDER BY 3 ASC
LIMIT 5;
````
#### Final Output

## 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
#### SQL Query
````sql
WITH filtered AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
  	HAVING COUNT(DISTINCT(month_year)) >= 6
)

SELECT
	id,
    interest_name,
    ROUND(STDDEV(percentile_ranking) :: DECIMAL, 2) AS percentile_ranking_deviation
FROM fresh_segments.interest_metrics AS t1
JOIN fresh_segments.interest_map AS t2
ON t1.interest_id :: INTEGER = t2.id
WHERE id IN (SELECT interest_id :: INTEGER FROM filtered)
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5;
````
#### Final Output

## 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
#### SQL Query
````sql
WITH filtered AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
  	HAVING COUNT(DISTINCT(month_year)) >= 6
),

interests AS (
	SELECT
		id,
    	interest_name,
    	ROUND(STDDEV(percentile_ranking) :: DECIMAL, 2) AS percentile_ranking_deviation
	FROM fresh_segments.interest_metrics AS t1
	JOIN fresh_segments.interest_map AS t2
	ON t1.interest_id :: INTEGER = t2.id
	WHERE id IN (SELECT interest_id :: INTEGER FROM filtered)
	GROUP BY 1, 2
	ORDER BY 3 DESC
	LIMIT 5
),

ranked_table AS (
	SELECT
  		id,
  		interest_name,
  		month_year,
  		percentile_ranking,
  		RANK() OVER(PARTITION BY id ORDER BY percentile_ranking ASC) AS min_percentile_rank,
  		RANK() OVER(PARTITION BY id ORDER BY percentile_ranking DESC) AS max_percentile_rank
	FROM fresh_segments.interest_metrics AS t1
	JOIN fresh_segments.interest_map AS t2
	ON t1.interest_id :: INTEGER = t2.id
  	WHERE id IN (SELECT id FROM interests)
)

SELECT
	id,
    interest_name,
    month_year,
    percentile_ranking
FROM ranked_table
WHERE min_percentile_rank = 1 OR max_percentile_rank = 1;
````
#### Final Output
