## 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
#### SQL Query
````sql
SELECT
	CASE WHEN registration_date >= '2021-01-01' AND registration_date <= '2021-01-01':: DATE + INTERVAL '6 days' THEN 1
    ELSE 2 END AS Week,
    COUNT(*)
FROM pizza_runner.runners
GROUP BY Week
ORDER BY Week;
````
#### Final Output
<img width="351" height="133" alt="image" src="https://github.com/user-attachments/assets/00518e0b-b432-41e9-a964-0ead1a18fb6d" />

## 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
#### SQL Query
````sql
SELECT 
    runner_id, 
    DATE_PART('minute', AVG(pickup_time :: timestamp - order_time)) AS avg_pickup_time
FROM customer_orders_clean
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY runner_id
ORDER BY runner_id ASC;
````
#### Final Output
<img width="264" height="134" alt="image" src="https://github.com/user-attachments/assets/2d7286cd-f61e-41f8-a87d-4f3c91398185" />

## 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
#### SQL Query
````sql
SELECT
	num_of_pizza,
    DATE_PART('minute', AVG(pickup_time :: timestamp - order_time) + INTERVAL '30 seconds') AS avg_prep_time
FROM (
	SELECT
		order_id,
  		order_time,
  		COUNT(*) AS num_of_pizza
	FROM customer_orders_clean
	GROUP BY order_id, order_time) AS t1
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY num_of_pizza
ORDER BY num_of_pizza;
````
#### Final Output
<img width="254" height="139" alt="image" src="https://github.com/user-attachments/assets/2c3fc52d-e81a-454e-b84a-a264d5749084" />

## 4. What was the average distance travelled for each customer?
#### SQL Query
````sql
SELECT 
    customer_id, 
    ROUND(AVG(distance :: DECIMAL),2) AS avg_distance
FROM (
    SELECT 
        DISTINCT(order_id), 
        customer_id
    FROM customer_orders_clean)
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE distance IS NOT NULL
GROUP BY customer_id
ORDER BY customer_id ASC;
````
#### Final Output
<img width="245" height="188" alt="image" src="https://github.com/user-attachments/assets/2d4289fc-8e9a-43f1-88ae-fcf40bf0e0a5" />

## 5. What was the difference between the longest and shortest delivery times for all orders?
#### SQL Query
````sql
SELECT 
    (MAX(duration :: DECIMAL) - MIN(duration :: DECIMAL)) AS difference
FROM runner_orders_clean
WHERE duration IS NOT NULL;
````
#### Final Output
<img width="85" height="75" alt="image" src="https://github.com/user-attachments/assets/bac61956-35ee-461f-8ec7-26274c1e3db7" />

## 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
#### SQL Query
````sql
SELECT
	order_id,
    runner_id,
    ROUND(distance :: DECIMAL / (duration :: DECIMAL / 60),2) AS avg_speed
FROM runner_orders_clean
WHERE distance IS NOT NULL
ORDER BY order_id;
````
#### Final Output
<img width="385" height="271" alt="image" src="https://github.com/user-attachments/assets/958599b9-ddbe-4093-96dc-dd161ce3fdbb" />

## 7. What is the successful delivery percentage for each runner?
#### SQL Query
````sql
SELECT
	runner_id,
    ROUND((successful_deliveries * 100 / total_deliveries :: DECIMAL)) AS delivery_percent
FROM (
 	SELECT
  		runner_id,
  		COUNT(*) AS total_deliveries
	FROM runner_orders_clean
 	GROUP BY runner_id) AS t1
LEFT JOIN (
	SELECT
  		runner_id,
  		COUNT(*) AS successful_deliveries
 	FROM runner_orders_clean
	WHERE pickup_time IS NOT NULL
  	GROUP BY runner_id) AS t2
USING(runner_id)
ORDER BY runner_id;
````
#### Final Output
<img width="265" height="131" alt="image" src="https://github.com/user-attachments/assets/af63c83d-d46b-4f82-b054-d4bb33dbee76" />
