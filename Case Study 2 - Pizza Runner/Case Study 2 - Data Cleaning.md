# Customer Orders Table
### Original Table
(picture of original table)

Removing any blank spaces or written 'null' values and actually making them NULL in the Exclusions column
````sql
UPDATE pizza_runner.customer_orders
SET exclusions = NULL
WHERE exclusions = 'null' OR exclusions = '';
````

Removing any blank spaces or written 'null' values and actually making them NULL in the Extras column
````sql
UPDATE pizza_runner.customer_orders
SET extras = NULL
WHERE extras = 'null' OR extras = '';
````

### Result of Data Cleaning for Customer Orders Table
````sql
SELECT *
FROM pizza_runner.customer_orders
````

# Runner Orders Table
### Original Table
(picture of original table)

Replacing all 'null' values with actual NULL in the distance column.
````sql
UPDATE pizza_runner.runner_orders
SET distance = NULL
WHERE distance = 'null';
````

Trimming all the km off so they can be treated as numbers in distance column
````sql
UPDATE pizza_runner.runner_orders
SET distance = TRIM('km' FROM distance);
````

Replace all 'null' values with NULL in pickup_time column
````sql
UPDATE pizza_runner.runner_orders
SET pickup_time = NULL
WHERE pickup_time = 'null';
````

Replace all 'null' values with NULL in duration column
````sql
UPDATE pizza_runner.runner_orders
SET duration = NULL
WHERE duration = 'null';
````

Trim all the words out of the duration column
````sql
UPDATE pizza_runner.runner_orders
SET duration = CASE
    WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
    ELSE duration
END;
````

Replace all 'null' or blank spaces with NULL in cancellation column
````sql
UPDATE pizza_runner.runner_orders
SET cancellation = NULL
WHERE cancellation = '' OR cancellation = 'null';
````

### Result of Data Cleaning for Runner Orders Table
````sql
TEST RUNNER_ORDERS
SELECT *
FROM pizza_runner.runner_orders;
````
