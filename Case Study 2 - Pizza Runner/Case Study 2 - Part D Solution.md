## 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
````sql
WITH profit AS (
	SELECT
  		CASE WHEN pizza_id = 1 THEN 12
  		ELSE 10 END AS costs
  	FROM pizza_runner.customer_orders
  	LEFT JOIN pizza_runner.runner_orders
  	USING(order_id)
  	WHERE pickup_time <> 'null'
)

SELECT
	SUM(costs) AS total_profits
FROM profit
````

## 2. What if there was an additional $1 charge for any pizza extras?
### Add cheese is $1 extra


## 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.


## 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas
## 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
````sql
WITH profits AS (
	SELECT
  		CASE WHEN pizza_id = 1 THEN 12
  		ELSE 10 END AS profit,
  		distance :: DECIMAL * 0.3 AS expenses
  	FROM pizza_runner.customer_orders
  	LEFT JOIN pizza_runner.runner_orders_clean
  	USING(order_id)
  	WHERE pickup_time IS NOT NULL
)

SELECT
	SUM(profit) - SUM(expenses) AS total_profit
FROM profits
````
