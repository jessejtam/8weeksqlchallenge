### 1. How many customers has Foodie-Fi ever had?
#### SQL Query
````sql
SELECT
	COUNT(DISTINCT(customer_id)) AS num_of_customers
FROM foodie_fi.subscriptions;
````
#### Final Output
<img width="128" height="75" alt="image" src="https://github.com/user-attachments/assets/658c1a72-a6e1-4c69-890f-d0d7e1675f3f" />

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
#### SQL Query
````sql
SELECT
	DATE_PART('month', start_date) AS month_name,
    COUNT(*) AS total_trials
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY month_name
ORDER BY month_name;
````
#### Final Output
<img width="230" height="374" alt="image" src="https://github.com/user-attachments/assets/4d618e21-ddab-44f8-804d-0f6d44969aa4" />

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
#### SQL Query
````sql
SELECT
	plan_name,
    COUNT(*) AS event_count
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING(plan_id)
WHERE start_date >= '2021-01-01'
GROUP BY plan_name;
````
#### Final Output
<img width="238" height="143" alt="image" src="https://github.com/user-attachments/assets/672316f6-2341-4f74-938e-69060b343cdf" />

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
#### SQL Query
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
#### Final Output
<img width="246" height="80" alt="image" src="https://github.com/user-attachments/assets/7c107af0-69e7-437a-8fc4-d6a48fe7d3ca" />

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
#### SQL Query
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
#### Final Output
<img width="240" height="79" alt="image" src="https://github.com/user-attachments/assets/de484b42-465f-43cc-8a1e-712b9386848a" />

### 6. What is the number and percentage of customer plans after their initial free trial?
#### SQL Query
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
	COUNT(*) AS plan_after_trial,
    ROUND(100 * COUNT(*) :: DECIMAL / 1000, 1) AS percentage
FROM ranked_table
LEFT JOIN foodie_fi.plans
USING(plan_id)
WHERE plan_rank = 2
GROUP BY plan_name;
````
#### Final Output
<img width="383" height="148" alt="image" src="https://github.com/user-attachments/assets/7359d978-2a4a-472d-b943-df8af03b917b" />

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
#### SQL Query
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
    COUNT(*) AS total_subscriptions,
    ROUND(100.0 * COUNT(*) / 1000, 2) AS percentage
FROM ranked_table AS t1
INNER JOIN max_rank AS t2
ON t1.customer_id = t2.customer_id AND t1.plan_rank = t2.top_rank
GROUP BY plan_name;
````
#### Final Output
<img width="413" height="168" alt="image" src="https://github.com/user-attachments/assets/4749ab72-7718-4013-943f-7a4b45adb7c2" />

### 8. How many customers have upgraded to an annual plan in 2020?
#### SQL Query
````sql
SELECT
	COUNT(DISTINCT(customer_id)) AS annual_plans
FROM foodie_fi.subscriptions
WHERE plan_id = 3 AND start_date <= '2020-12-31';
````
#### Final Output
<img width="100" height="74" alt="image" src="https://github.com/user-attachments/assets/83e4750c-2a67-422c-adc2-feae69273d9f" />

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
#### SQL Query
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
#### Final Output
<img width="89" height="78" alt="image" src="https://github.com/user-attachments/assets/fe96d5e9-e442-48d1-aa0d-b1a50d69a927" />

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
#### SQL Query
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
  	WHERE annual IS NOT NULL AND trial IS NOT NULL
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
#### Final Output
<img width="229" height="334" alt="image" src="https://github.com/user-attachments/assets/2b0da0bc-3e25-4314-8963-46ddd3b3598f" />

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
#### SQL Query
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
#### Final Output
<img width="94" height="76" alt="image" src="https://github.com/user-attachments/assets/a7e5d1cb-1c2f-4ebc-84b4-0ba61ffbe2b6" />
