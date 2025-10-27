## 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
````sql
SELECT
	CASE WHEN registration_date >= '2021-01-01' AND registration_date <= '2021-01-01':: DATE + INTERVAL '6 days' THEN 1
    ELSE 2 END AS Week,
    COUNT(*)
FROM pizza_runner.runners
GROUP BY Week
ORDER BY Week;
````

## 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
````sql
SELECT
	runner_id,
    DATE_TRUNC('second', AVG(pickup_time :: TIMESTAMP - order_time)) AS avg_time
FROM pizza_runner.customer_orders
LEFT JOIN pizza_runner.runner_orders
USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY runner_id
ORDER BY runner_id;
````

## 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?


## 4. What was the average distance travelled for each customer?
DONE

## 5. What was the difference between the longest and shortest delivery times for all orders?
DONE

## 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?


## 7. What is the successful delivery percentage for each runner?
