### NEED TO WORK ON AND UNDERSTAND MORE
CREATE TABLE payments AS
	SELECT
    	customer_id,
        plan_id,
        plan_name,
        GENERATE_SERIES(
        	start_date,
          	CASE WHEN ,
          	INTERVAL '1 MONTH'
        ) :: DATE AS payment_date,
        price AS amount,
        RANK() OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS payment_order
    FROM foodie_fi.subscriptions
    LEFT JOIN foodie_fi.plans
    USING(plan_id)
    WHERE plan_id <> 0;
    
SELECT * FROM payments
