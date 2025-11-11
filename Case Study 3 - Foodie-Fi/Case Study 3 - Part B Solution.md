### 1. How many customers has Foodie-Fi ever had?
````sql
SELECT
	COUNT(DISTINCT(customer_id))
FROM foodie_fi.subscriptions;
````

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
````sql
SELECT
	TO_CHAR(start_date, 'Month') AS month_name,
    COUNT(*) AS total_trials
FROM foodie_fi.subscriptions
WHERE plan_id = 1
GROUP BY month_name
ORDER BY total_trials DESC;
````

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
````sql
SELECT
	plan_name,
    COUNT(*)
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING(plan_id)
WHERE start_date >= '2021-01-01'
GROUP BY plan_name;
````

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
````sql
WITH churned AS (
	SELECT
  		COUNT(DISTINCT(customer_id)) AS total_churn
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 4
)

SELECT
	total_churn,
	ROUND(100 * total_churn :: DECIMAL / 1000, 1) AS percent_churn
FROM churned
````

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
````sql
WITH ranked_table AS (
	SELECT
  		customer_id,
  		plan_id,
  		start_date,
  		RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS plan_rank
  	FROM foodie_fi.subscriptions
)

SELECT
	COUNT(*) AS churn_after_trial,
    100 * COUNT(*) / 1000 AS percentage
FROM ranked_table
WHERE plan_rank = 2 AND plan_id = 4;
````

### 6. What is the number and percentage of customer plans after their initial free trial?
````sql
WITH ranked_table AS (
	SELECT
  		customer_id,
  		plan_id,
  		start_date,
  		RANK() OVER(PARTITION BY customer_id ORDER BY start_date) AS plan_rank
  	FROM foodie_fi.subscriptions
)

SELECT
	plan_name,
	COUNT(*) AS plans_after_trial,
    ROUND(100 * COUNT(*) :: DECIMAL / 1000, 1) AS percentage
FROM ranked_table
LEFT JOIN foodie_fi.plans
USING(plan_id)
WHERE plan_rank = 2
GROUP BY plan_name;
````

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
````sql
WITH ranked_table AS (
	SELECT
  		customer_id,
  		plan_name,
  		start_date,
  		RANK() OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS plan_rank
  	FROM foodie_fi.subscriptions
  	LEFT JOIN foodie_fi.plans
  	USING(plan_id)
  	WHERE start_date <= '2020-12-31'
),

max_rank AS (
	SELECT 
  		customer_id,
  		MAX(plan_rank) AS top_rank
  	FROM ranked_table
  	GROUP BY customer_id
)

SELECT
	plan_name,
    COUNT(*) AS total_subscriptions
FROM ranked_table AS t1
INNER JOIN max_rank AS t2
ON t1.customer_id = t2.customer_id AND t1.plan_rank = t2.top_rank
GROUP BY plan_name;
````

### 8. How many customers have upgraded to an annual plan in 2020?
````sql
SELECT
	COUNT(DISTINCT(customer_id)) AS annual_plans
FROM foodie_fi.subscriptions
WHERE plan_id = 3 AND start_date <= '2020-12-31';
````

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
````sql
WITH trial_date AS (
  	SELECT
  		customer_id,
  		start_date
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 0
),

annual_date AS (
	SELECT
  		customer_id,
  		start_date
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 3
)

SELECT
	ROUND(AVG(a.start_date - t.start_date),0) AS avg_days
FROM trial_date AS t
LEFT JOIN annual_date AS a
USING(customer_id);
````

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc) NEED TO FIX
````sql
WITH trial_date AS (
  	SELECT
  		customer_id,
  		start_date AS trial
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 0
),

annual_date AS (
	SELECT
  		customer_id,
  		start_date AS annual
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 3
),

day_difference AS (
	SELECT
  		customer_id,
  		annual - trial AS difference
  	FROM trial_date
  	LEFT JOIN annual_date
  	USING(customer_id)
)

SELECT
	CASE WHEN difference BETWEEN 0 AND 30 THEN '0-30'
    	WHEN difference BETWEEN 31 AND 60 THEN '31-60'
    	WHEN difference BETWEEN 61 AND 90 THEN '61-90'
    	WHEN difference BETWEEN 91 AND 120 THEN '91-120'
    	WHEN difference BETWEEN 121 AND 150 THEN '121-150'
    	WHEN difference BETWEEN 151 AND 180 THEN '151-180'
    	WHEN difference BETWEEN 181 AND 210 THEN '181-210'
    	WHEN difference BETWEEN 211 AND 240 THEN '211-240'
    	WHEN difference BETWEEN 241 AND 270 THEN '241-270'
    	WHEN difference BETWEEN 271 AND 300 THEN '271-300'
    	WHEN difference BETWEEN 301 AND 330 THEN '301-330'
    ELSE '331-360' END AS time_period,
    COUNT(*)
FROM day_difference
GROUP BY time_period
ORDER BY MIN(difference);
````

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
````sql
WITH ranked_table AS (
	SELECT
  		customer_id,
  		plan_id,
  		start_date,
  		RANK() OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS date_rank
  	FROM foodie_fi.subscriptions
  	WHERE plan_id = 1 OR plan_id = 2 AND start_date <= '2020-12-31'
)

SELECT
	COUNT(*) AS downgraded
FROM ranked_table
WHERE plan_id = 2 AND date_rank = 1 AND plan_id = 1 AND date_rank = 2
````
