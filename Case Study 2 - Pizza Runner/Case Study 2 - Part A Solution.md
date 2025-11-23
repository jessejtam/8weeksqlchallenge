## 1. How many pizzas were ordered?
#### SQL Query
````sql
SELECT 
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean;
````
#### Final Output
<img width="105" height="75" alt="image" src="https://github.com/user-attachments/assets/33467c2a-8218-49e4-8e4c-15a2f47b7240" />

## 2. How many unique customer orders were made?
#### SQL Query
````sql
SELECT 
    COUNT(DISTINCT(order_time)) AS unique_customer_orders
FROM customer_orders_clean;
````
#### Final Output
<img width="164" height="69" alt="image" src="https://github.com/user-attachments/assets/e0c014a0-62bf-4583-b365-85773138292e" />

## 3. How many successful orders were delivered by each runner?
#### SQL Query
````sql
SELECT 
    runner_id, 
    COUNT(pickup_time) AS successful_deliveries
FROM runner_orders_clean
WHERE pickup_time IS NOT NULL
GROUP BY runner_id
ORDER BY runner_id;
````
#### Final Output
<img width="288" height="130" alt="image" src="https://github.com/user-attachments/assets/c1dc9b68-7a42-45df-8791-d61bb97d2885" />

## 4. How many of each type of pizza was delivered?
#### SQL Query
````sql
SELECT 
    pizza_name, 
    COUNT(pizza_name) AS pizzas_delivered
FROM customer_orders_clean
LEFT JOIN runner_orders_clean
USING(order_id)
LEFT JOIN pizza_runner.pizza_names
USING(pizza_id)
WHERE pickup_time IS NOT NULL
GROUP BY pizza_name;
````
#### Final Output
<img width="265" height="102" alt="image" src="https://github.com/user-attachments/assets/84bbf9de-e3bb-4772-abd5-1d6869b9f28e" />

## 5. How many Vegetarian and Meatlovers were ordered by each customer?
#### SQL Query
````sql
SELECT
    customer_id,
    SUM(CASE WHEN pizza_name = 'Meatlovers' THEN 1 ELSE 0 END) AS number_of_meatlovers,
    SUM(CASE WHEN pizza_name = 'Vegetarian' THEN 1 ELSE 0 END) AS number_of_vegetarian
FROM customer_orders_clean
LEFT JOIN pizza_runner.pizza_names
USING(pizza_id)
GROUP BY customer_id
ORDER BY customer_id;
````
#### Final Output
<img width="480" height="190" alt="image" src="https://github.com/user-attachments/assets/e7f2a52c-276b-4d79-b323-53e465587394" />

## 6. What was the maximum number of pizzas delivered in a single order?
#### SQL Query
````sql
SELECT
    MAX(pizzas_delivered) AS pizzas_delivered
FROM (
    SELECT 
        order_id, 
        COUNT(*) AS pizzas_delivered
    FROM customer_orders_clean
    LEFT JOIN runner_orders_clean
    USING(order_id)
    WHERE pickup_time IS NOT NULL
    GROUP BY order_id)
````
#### Final Output
<img width="113" height="75" alt="image" src="https://github.com/user-attachments/assets/8300ba58-3719-43f7-88e6-28aa6a9947d7" />

## 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
#### SQL Query
````sql
WITH changes AS (
  SELECT 
    customer_id,
    CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1
        ELSE 0 END AS unchanged_pizzas,
    CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
        ELSE 0 END AS changed_pizzas
    FROM customer_orders_clean
    LEFT JOIN runner_orders_clean
    USING(order_id)
    WHERE distance IS NOT NULL)
   
SELECT 
    customer_id, 
    SUM(changed_pizzas) AS changed_pizzas, 
    SUM(unchanged_pizzas) AS unchanged_pizzas
FROM changes
GROUP BY customer_id
ORDER BY customer_id;
````
#### Final Output
<img width="422" height="189" alt="image" src="https://github.com/user-attachments/assets/3af8f7cd-0b11-4125-9d96-ff2dcb6b5bf2" />

## 8. How many pizzas were delivered that had both exclusions and extras?
#### SQL Query
````sql
SELECT 
    COUNT(*) AS pizza_count
FROM customer_orders_clean
LEFT JOIN runner_orders_clean
USING(order_id)
WHERE exclusions IS NOT NULL AND extras IS NOT NULL AND distance IS NOT NULL;
````
#### Final Output
<img width="85" height="77" alt="image" src="https://github.com/user-attachments/assets/3b23de62-eda3-4b6d-9e7b-071ad4a00234" />

## 9. What was the total volume of pizzas ordered for each hour of the day?
#### SQL Query
````sql
SELECT 
    EXTRACT(hour FROM order_time) AS hour, 
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean
GROUP BY hour
ORDER BY hour;
````
#### Final Output
<img width="255" height="191" alt="image" src="https://github.com/user-attachments/assets/bdbbe3c4-4d95-473b-854b-35c38adc0775" />

## 10. What was the volume of orders for each day of the week?
#### SQL Query
````sql
SELECT 
    TO_CHAR(order_time, 'DAY') AS day_of_the_week, 
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean
GROUP BY day_of_the_week;
````
#### Final Output
<img width="258" height="153" alt="image" src="https://github.com/user-attachments/assets/5da938e8-a228-4482-aaea-e93b05aad192" />
