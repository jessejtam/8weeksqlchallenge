# Customer Orders Table
#### Original Table
<img width="894" height="429" alt="image" src="https://github.com/user-attachments/assets/02658194-5628-46b5-873e-89cf193aad5d" />

Removing any blank spaces or written 'null' values and actually making them NULL in the Exclusions column
#### SQL Query
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
#### Result of Data Cleaning for Customer Orders Table
````sql
SELECT *
FROM customer_orders_clean
````
<img width="889" height="431" alt="image" src="https://github.com/user-attachments/assets/4c828e12-effa-4578-8cf9-87229f8ea0b5" />

# Runner Orders Table
#### Original Table
<img width="910" height="333" alt="image" src="https://github.com/user-attachments/assets/3aedcfc7-0a78-42e1-95cb-dc9719945107" />

Replacing all 'null' values with actual NULL in the distance column.
#### SQL Query
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
<img width="904" height="326" alt="image" src="https://github.com/user-attachments/assets/f68fe80e-3784-4343-8a73-32ef6aa4b581" />
