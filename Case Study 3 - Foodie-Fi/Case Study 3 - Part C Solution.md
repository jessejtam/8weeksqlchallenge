## The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

#### SQL Query
````sql
CREATE TABLE payments AS
WITH plan_dates AS (
	SELECT
  		customer_id,
  		plan_id,
  		plan_name,
        start_date,
  		LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY start_date) AS end_date,
  		LEAD(price) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_price,
  		price AS amount
	FROM foodie_fi.subscriptions
  	LEFT JOIN foodie_fi.plans
  	USING(plan_id)
  	WHERE plan_id <> 0 AND customer_id IN (1, 2, 11, 13, 15, 16, 18, 19) AND start_date <= '2020-12-31'
),

series AS (
	SELECT
  		customer_id,
  		plan_id,
  		plan_name,
  		generate_series(
        	start_date,
          	CASE WHEN end_date IS NULL THEN '2021-1-1'
          	ELSE end_date - 1 END,
          	CASE WHEN plan_id = 1 OR plan_id = 2 THEN INTERVAL '1 month'
      		ELSE INTERVAL '1 year' END
        ) :: DATE AS payment_date,
  		amount
  	FROM plan_dates
  	WHERE plan_id <> 4
)
SELECT
	customer_id,
    plan_id,
    plan_name,
    payment_date,
    CASE WHEN LAG(plan_id) OVER(PARTITION BY customer_id ORDER BY payment_date) = 1 AND payment_date < LAG(payment_date) OVER(PARTITION BY customer_id ORDER BY payment_date) + INTERVAL '1 month' THEN amount - 9.90
    	WHEN LAG(plan_id) OVER(PARTITION BY customer_id ORDER BY payment_date) = 2 AND payment_date < LAG(payment_date) OVER(PARTITION BY customer_id ORDER BY payment_date) + INTERVAL '1 month' THEN amount - 19.90
   	ELSE amount END,
    RANK() OVER(PARTITION BY customer_id ORDER BY payment_date) AS payment_order
FROM series;

SELECT *
FROM payments
````
#### Final Output
<img width="859" height="696" alt="image" src="https://github.com/user-attachments/assets/a57f72fa-954f-4680-951c-874382d217dc" />
