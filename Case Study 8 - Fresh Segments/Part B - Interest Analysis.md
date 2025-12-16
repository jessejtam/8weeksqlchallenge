# Part B - Interest Analysis
## 1. Which interests have been present in all month_year dates in our dataset?
#### SQL Query
````sql
SELECT
	id,
    interest_name
FROM fresh_segments.interest_metrics AS t1
LEFT JOIN fresh_segments.interest_map AS t2
ON t1.interest_id :: INTEGER = t2.id
GROUP BY 1, 2
HAVING STRING_AGG(DISTINCT month_year :: TEXT, ',') =
  (SELECT STRING_AGG(DISTINCT month_year :: TEXT, ',') FROM fresh_segments.interest_metrics)
````
#### Final Output

## 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
#### SQL Query
````sql
WITH t1 AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
),

t2 AS (
	SELECT
  		total_months,
  		COUNT(*) AS count
  	FROM t1
 	GROUP BY 1
)

SELECT
	total_months,
    ROUND(100.0 * SUM(count) OVER(ORDER BY total_months DESC) / (SELECT SUM(count) FROM t2), 2) AS cumulative_percent
FROM t2
GROUP BY total_months, count;
````
#### Final Output

## 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
#### SQL Query
````sql
WITH t1 AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
)

SELECT
	COUNT(*) AS datapoints_removed
FROM t1
WHERE total_months < 14;
````
#### Final Output

## 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
No, it does not make sense to remove these data points from a business perspective because by removing them, you would be removing over half of the dataset.

## 5. After removing these interests - how many unique interests are there for each month?
#### SQL Query
````sql
WITH t1 AS (
	SELECT
  		interest_id,
  		COUNT(DISTINCT(month_year)) AS total_months
  	FROM fresh_segments.interest_metrics
  	GROUP BY 1
)

SELECT
	COUNT(DISTINCT(interest_id)) AS unique_interests
FROM t1
WHERE total_months = 14;
````
#### Final Output
