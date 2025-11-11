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


### 8. How many customers have upgraded to an annual plan in 2020?
````sql
SELECT
	COUNT(DISTINCT(customer_id)) AS annual_plans
FROM foodie_fi.subscriptions
WHERE plan_id = 3 AND start_date <= '2020-12-31';
````

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?


### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)


### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

