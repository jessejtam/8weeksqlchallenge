-- PART A: Pizza Metrics
## 1. How many pizzas were ordered?
````sql
SELECT 
    COUNT(*) AS pizzas_ordered
FROM pizza_runner.customer_orders
````

## 2. How many unique customer orders were made?
````sql
SELECT 
    COUNT(DISTINCT(order_time)) AS unique_customer_orders
FROM pizza_runner.customer_orders;
````

## 3. How many successful orders were delivered by each runner?
````sql
SELECT 
    runner_id, 
    COUNT(pickup_time) AS successful_deliveries
FROM pizza_runner.runner_orders
WHERE pickup_time <> 'null'
GROUP BY runner_id
ORDER BY runner_id;
````

## 4. How many of each type of pizza was delivered?
````sql
SELECT 
    pizza_name, 
    COUNT(pizza_name) AS pizzas_delivered
FROM pizza_runner.customer_orders
LEFT JOIN pizza_runner.runner_orders
USING(order_id)
LEFT JOIN pizza_runner.pizza_names
USING(pizza_id)
WHERE pickup_time <> 'null'
GROUP BY pizza_name;
````

## 5. How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT 
    customer_id, 
    pizza_name, 
    COUNT(*) AS pizzas_ordered
FROM pizza_runner.customer_orders
LEFT JOIN pizza_runner.pizza_names
USING(pizza_id)
GROUP BY customer_id, pizza_name
ORDER BY customer_id;
````

## 6. What was the maximum number of pizzas delivered in a single order?
````sql
SELECT
    MAX(pizzas_delivered) AS pizzas_delivered
FROM (SELECT 
        order_id, 
        COUNT(*) AS pizzas_delivered
    FROM pizza_runner.customer_orders
    LEFT JOIN pizza_runner.runner_orders
    USING(order_id)
    WHERE pickup_time <> 'null'
    GROUP BY order_id)
````

## 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
WITH changes AS (
  SELECT 
    customer_id,
    CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1
        ELSE 0 END AS unchanged_pizzas,
    CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
        ELSE 0 END AS changed_pizzas
    FROM pizza_runner.customer_orders
    LEFT JOIN pizza_runner.runner_orders
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

## 8. How many pizzas were delivered that had both exclusions and extras?
````sql
SELECT 
    COUNT(*) AS pizza_count
FROM pizza_runner.customer_orders
LEFT JOIN pizza_runner.runner_orders
USING(order_id)
WHERE exclusions IS NOT NULL AND extras IS NOT NULL AND distance IS NOT NULL;
````

## 9. What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT 
    EXTRACT(hour FROM order_time) AS hour, 
    COUNT(*) AS pizzas_ordered
FROM pizza_runner.customer_orders
GROUP BY hour
ORDER BY hour;
````

## 10. What was the volume of orders for each day of the week?
````sql
SELECT 
    TO_CHAR((order_time + INTERVAL '2 days'), 'DAY') AS day_of_the_week, 
    COUNT(*) AS pizzas_ordered
FROM pizza_runner.customer_orders
GROUP BY day_of_the_week
ORDER BY pizzas_ordered DESC;
````
