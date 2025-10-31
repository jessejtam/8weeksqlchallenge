# Customer Orders Table
### Original Table
(picture of original table)

Removing any blank spaces or written 'null' values and actually making them NULL in the Exclusions column
````sql
CREATE TABLE customer_orders_clean AS
SELECT
	order_id,
    customer_id,
    pizza_id,
	CASE WHEN exclusions = '' OR exclusions = 'null' THEN NULL
    ELSE exclusions END AS exclusions,
	CASE WHEN extras = '' OR extras = 'null' THEN NULL
    ELSE extras END AS extras,
    order_time
FROM pizza_runner.customer_orders;
````

### Result of Data Cleaning for Customer Orders Table
````sql
SELECT *
FROM customer_orders_clean
````

# Runner Orders Table
### Original Table
(picture of original table)

Replacing all 'null' values with actual NULL in the distance column.
````sql
CREATE TABLE runner_orders_clean AS
SELECT
	order_id,
    runner_id,
    CASE WHEN pickup_time = 'null' THEN NULL
    ELSE pickup_time END AS pickup_time,
    CASE WHEN distance = 'null' THEN NULL
    ELSE TRIM('km' FROM distance) END AS distance,
    CASE WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
        WHEN duration = 'null' THEN NULL
    ELSE duration END AS duration,
    CASE WHEN cancellation = '' OR cancellation = 'null' THEN NULL
    ELSE cancellation END AS cancellation
FROM pizza_runner.runner_orders;
````

### Result of Data Cleaning for Runner Orders Table
````sql
SELECT *
FROM pizza_runner.runner_orders_clean
````
