# Part A - Data Exploration and Cleansing
## 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
#### SQL Query
````sql
ALTER TABLE fresh_segments.interest_metrics
ALTER COLUMN month_year TYPE DATE USING TO_DATE(month_year, 'MM/YYYY');
````
#### Final Output

## 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
#### SQL Query
````sql
SELECT
	month_year,
    COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY 1
ORDER BY 1 NULLS FIRST;
````
#### Final Output

## 3. What do you think we should do with these null values in the fresh_segments.interest_metrics
I think that we should remove all the null values in the fresh_segment.interest_metrics table because they don't give us any usable information when there is no month or year to attach the data to.
#### SQL Query
````sql
DELETE FROM fresh_segments.interest_metrics
WHERE month_year IS NULL;
````
#### Final Output

## 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
#### SQL Query
````sql
WITH t1 AS (
	SELECT
  		COUNT(*) AS metrics_exclusive_id
  	FROM fresh_segments.interest_metrics
  	WHERE interest_id :: INTEGER NOT IN (SELECT id FROM fresh_segments.interest_map)
),

t2 AS (
	SELECT
  		COUNT(*) AS map_exclusive_id
  	FROM fresh_segments.interest_map
  	WHERE id NOT IN (SELECT interest_id :: INTEGER FROM fresh_segments.interest_metrics)
)

SELECT *
FROM t1
CROSS JOIN t2;
````
#### Final Output

## 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table
#### SQL Query
````sql
SELECT
	id,
    interest_name,
    COUNT(*)
FROM fresh_segments.interest_map AS t1
JOIN fresh_segments.interest_metrics AS t2
ON t1.id = t2.interest_id :: INTEGER
GROUP BY 1, 2
ORDER BY 3 DESC;
````
#### Final Output

## 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
We should perform an inner join since interest_map has 7 exclusive ids and we don't want to include them since nobody uses them anyways. We also don't want to worry about the order we join them in (aka which table is the first table) so an inner join simplifies this.
#### SQL Query
````sql
SELECT
    _month,
    _year,
    month_year,
   	interest_id,
    composition,
    index_value,
    ranking,
    percentile_ranking,
    interest_name,
    interest_summary,
    created_at,
    last_modified
FROM fresh_segments.interest_map AS t1
JOIN fresh_segments.interest_metrics AS t2
ON t1.id = t2.interest_id :: INTEGER
WHERE interest_id = '21246'
ORDER BY month_year;
````
#### Final Output

## 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
#### SQL Query
````sql
WITH before_create AS (
	SELECT
		COUNT(*) AS ids_created_before
	FROM fresh_segments.interest_map AS t1
	JOIN fresh_segments.interest_metrics AS t2
	ON t1.id = t2.interest_id :: INTEGER
	WHERE month_year < created_at
),

truncated AS (
	SELECT
		COUNT(*) AS same_month
	FROM fresh_segments.interest_map AS t1
	JOIN fresh_segments.interest_metrics AS t2
	ON t1.id = t2.interest_id :: INTEGER
	WHERE month_year < created_at AND month_year = DATE_TRUNC('month', created_at)
)

SELECT *
FROM before_create
CROSS JOIN truncated;
````
#### Final Output

Yes, there are 188 records where month_year is before the created_value, however, this is because we assumed month_year to be the start of every month, and when we truncate the date from the created_at date, we see that the years and months for both the month_year and created_at values are the same.
