## 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
#### SQL Query
````sql
SELECT
	SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS total_profit
FROM customer_orders_clean
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE pickup_time IS NOT NULL;
````
#### Final Output

## 2. What if there was an additional $1 charge for any pizza extras?
#### Add cheese is $1 extra
#### SQL Query
````sql
WITH base AS (
	SELECT
		SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS pizza_cost
	FROM customer_orders_clean
	LEFT JOIN runner_orders_clean
	USING(order_id)
	WHERE pickup_time IS NOT NULL
),

extras AS (
	SELECT
  		SUM(CASE WHEN extras = 4 THEN 2 ELSE 1 END) AS extras_cost
  	FROM (
    	SELECT
      		REGEXP_SPLIT_TO_TABLE(extras, ',') :: INTEGER AS extras
      	FROM customer_orders_clean
      	LEFT JOIN runner_orders_clean
      	USING(order_id)
      	WHERE pickup_time IS NOT NULL
    ) AS t1	
)

SELECT
	pizza_cost + extras_cost AS total_profit
FROM base
CROSS JOIN extras;
````
#### Final Output

## 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
#### SQL Query
````sql
CREATE TABLE rating_table AS
SELECT
	order_id,
    runner_id,
    distance :: DECIMAL AS delivery_distance,
    duration :: DECIMAL AS delivery_time,
    NULL :: DECIMAL AS rating,
    NULL :: VARCHAR AS comments
FROM customer_orders_clean
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY order_id, runner_id, delivery_distance, delivery_time, rating, comments
ORDER BY order_id;

UPDATE rating_table
SET rating = 
		CASE WHEN order_id = 1 THEN 3.5
    		WHEN order_id = 2 THEN 4
        	WHEN order_id = 3 THEN 5
        	WHEN order_id = 4 THEN 1
        	WHEN order_id = 5 THEN 5
        	WHEN order_id = 7 THEN 5
        	WHEN order_id = 8 THEN 5
        	WHEN order_id = 10 THEN 5
    	END,
    comments =
    	CASE WHEN order_id = 1 THEN 'Food took a long time to come'
            WHEN order_id = 2 THEN 'Food took a while to come but everything was there and warm'
        	WHEN order_id = 3 THEN 'Food was nice and warm and person was respectful'
        	WHEN order_id = 4 THEN 'THEY WERE SLOW AND ATE ONE OF MY SLICES!!!'
        	WHEN order_id = 5 THEN 'Pizza was hot and delivery was quick'
        	WHEN order_id = 7 THEN 'Good'
        	WHEN order_id = 8 THEN 'Very worrying how fast they delivered my pizza...'
        	WHEN order_id = 10 THEN 'Yes'
        END;
````
#### Final Output

## 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
#### SQL Query
````sql
SELECT
	customer_id,
    order_id,
    t2.runner_id,
    rating,
    order_time,
    pickup_time,
    pickup_time :: TIMESTAMP - order_time AS time_between_order_and_pickup,
    delivery_time AS delivery_duration,
    ROUND(60.0 * delivery_distance / delivery_time, 2) AS avg_speed,
    COUNT(*) AS num_of_pizzas
FROM customer_orders_clean AS t1
LEFT JOIN runner_orders_clean AS t2
USING(order_id)
LEFT JOIN rating_table AS t3
USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY 1, 2, 3, 4, 5, 6, 7, 8, 9
ORDER BY order_id;
````
#### Final Output

## 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
#### SQL Query
````sql
WITH runner_cost AS (
	SELECT
  		0.3 * SUM(distance :: DECIMAL) AS runner_costs
  	FROM runner_orders_clean
),

profits AS (
	SELECT
  		SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS profit
  	FROM customer_orders_clean
  	LEFT JOIN runner_orders_clean
  	USING(order_id)
  	WHERE pickup_time IS NOT NULL
)

SELECT
	profit - runner_costs AS total_profits
FROM runner_cost
CROSS JOIN profits;
````
#### Final Output
