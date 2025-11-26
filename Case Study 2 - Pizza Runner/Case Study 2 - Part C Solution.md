# Part C - Ingredient Optimization
## 1. What are the standard ingredients for each pizza?
#### SQL Query
````sql
WITH ingredients AS (
	SELECT 
		pizza_name,
    	topping_name
	FROM (
  		SELECT 
			pizza_id, 
    		REGEXP_SPLIT_TO_TABLE(toppings, ',') :: INTEGER AS tops
  		FROM pizza_recipes) AS t1
	LEFT JOIN pizza_toppings AS t2
	ON t1.tops = t2.topping_id
  	LEFT JOIN pizza_names
  	USING(pizza_id)
	ORDER BY pizza_id, tops)

SELECT 
	pizza_name,
    STRING_AGG(topping_name, ', ') AS toppings
FROM ingredients
GROUP BY pizza_name;
````
#### Final Output
<img width="590" height="107" alt="image" src="https://github.com/user-attachments/assets/23119aa0-12af-40b8-8d90-2548eb1b257a" />

## 2. What was the most commonly added extra?
#### SQL Query
````sql
WITH toppings AS (
	SELECT
  		REGEXP_SPLIT_TO_TABLE(extras, ',') AS extra
  	FROM customer_orders_clean
)

SELECT
	topping_name,
    COUNT(extra) AS times_added
FROM toppings AS t1
LEFT JOIN pizza_runner.pizza_toppings AS t2
ON t1.extra :: INTEGER = t2.topping_id
GROUP BY topping_name
ORDER BY times_added DESC
LIMIT 1;
````
#### Final Output
<img width="246" height="79" alt="image" src="https://github.com/user-attachments/assets/eb4e9507-eecc-4041-9f1f-090407c2ef38" />

## 3. What was the most common exclusion?
#### SQL Query
````sql
WITH toppings AS (
	SELECT
  		REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS exclusion
  	FROM customer_orders_clean
)

SELECT
	topping_name,
    COUNT(exclusion) AS times_excluded
FROM toppings AS t1
LEFT JOIN pizza_runner.pizza_toppings AS t2
ON t1.exclusion :: INTEGER = t2.topping_id
GROUP BY topping_name
ORDER BY times_excluded DESC
LIMIT 1;
````
#### Final Output
<img width="257" height="79" alt="image" src="https://github.com/user-attachments/assets/5f25b316-bb9f-4a59-bc72-618b24bd8d52" />

## 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
### - Meat Lovers
### - Meat Lovers - Exclude Beef
### - Meat Lovers - Extra Bacon
### - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
#### SQL Query
````sql
WITH exclusion AS (
	SELECT
  		order_id,
  		STRING_AGG(topping_name, ', ') AS exclude_topping,
  		rn
  	FROM (
  		SELECT
  			order_id,
      		ROW_NUMBER() OVER(ORDER BY order_id ASC, pizza_id ASC, exclusions DESC) AS rn,
  			REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS topping
  		FROM customer_orders_clean) AS t1 
  	LEFT JOIN pizza_runner.pizza_toppings AS t2
  	ON t1.topping :: INTEGER = t2.topping_id
  	GROUP BY order_id, rn
),

extra AS (
	SELECT
  		order_id,
  		STRING_AGG(topping_name, ', ') AS extra_topping,
  		rn
  	FROM (
  		SELECT
  			order_id,
      		ROW_NUMBER() OVER(ORDER BY order_id ASC, pizza_id ASC, exclusions DESC) AS rn,
  			REGEXP_SPLIT_TO_TABLE(extras, ',') AS topping
  		FROM customer_orders_clean) AS t1 
  	LEFT JOIN pizza_runner.pizza_toppings AS t2
  	ON t1.topping :: INTEGER = t2.topping_id
  	GROUP BY order_id, rn	
),

words AS (
	SELECT
  		order_id,
  		pizza_name,
  		ROW_NUMBER() OVER(ORDER BY order_id ASC, pizza_id ASC, exclusions DESC) AS rn,
  		CASE WHEN exclusions IS NULL THEN ' '
  		ELSE ' - Exclude ' END AS exclude_word,
  		CASE WHEN extras IS NULL THEN ' '
  		ELSE ' - Extra ' END AS extra_word
  	FROM customer_orders_clean
  	LEFT JOIN pizza_runner.pizza_names
  	USING(pizza_id)
)

SELECT
	words.order_id,
	CONCAT(pizza_name, exclude_word, exclude_topping, extra_word, extra_topping) AS orders
FROM words
LEFT JOIN exclusion
ON words.order_id = exclusion.order_id AND words.rn = exclusion.rn
LEFT JOIN extra
ON words.order_id = extra.order_id AND words.rn = extra.rn
````
#### Final Output
<img width="569" height="430" alt="image" src="https://github.com/user-attachments/assets/3fb14858-3f32-400d-bd1f-73820330941e" />

## 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
#### SQL Query
````sql
WITH pizza_table AS (
	SELECT
  		order_id,
  		pizza_name,
  		pizza_id,
  		exclusions,
  		extras,
  		ROW_NUMBER() OVER(ORDER BY order_id ASC, exclusions DESC) AS row_num
  	FROM customer_orders_clean
  	LEFT JOIN pizza_runner.pizza_names
  	USING(pizza_id)
),

regular AS (
	SELECT
  		order_id,
  		toppings,
  		COUNT(toppings) AS total,
  		row_num
  	FROM (
      	SELECT
      		order_id,
      		REGEXP_SPLIT_TO_TABLE(toppings, ',') AS toppings,
      		row_num
      	FROM pizza_table
      	LEFT JOIN pizza_runner.pizza_recipes
     	USING(pizza_id)
    ) AS t1
  	GROUP BY toppings, order_id, row_num
),

exclusion AS (
	SELECT
  		order_id,
  		toppings,
  		COUNT(toppings) * -1,
  		row_num
  	FROM (
    	SELECT
      		order_id,
      		REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS toppings,
      		row_num
      	FROM pizza_table
    ) AS t1
  	GROUP BY toppings, order_id, row_num
),

extra AS (
	SELECT
  		order_id,
  		toppings,
  		COUNT(toppings),
  		row_num
  	FROM (
    	SELECT
      		order_id,
      		REGEXP_SPLIT_TO_TABLE(extras, ',') AS toppings,
      		row_num
      	FROM pizza_table
    ) AS t1
  	GROUP BY toppings, order_id, row_num
),

combine AS (
	SELECT
  		order_id,
  		topping_name,
  		SUM(total) AS total,
  		row_num
  	FROM (
    	SELECT *
  			FROM regular
            UNION ALL
            SELECT *
            FROM exclusion
            UNION ALL
            SELECT *
            FROM extra
    ) AS t1
  	LEFT JOIN pizza_runner.pizza_toppings AS t2
  	ON t1.toppings :: INTEGER = t2.topping_id
  	GROUP BY topping_name, order_id, row_num
),

wording AS (
	SELECT
		order_id,
  		STRING_AGG(topping_name, ', ' ORDER BY topping_name ASC) AS ingredients,
  		row_num
  	FROM (
      	SELECT
      		order_id,
      		CASE WHEN total = 0 THEN NULL
  			WHEN total = 2 THEN CONCAT('2x', topping_name)
  			ELSE topping_name END AS topping_name,
      		row_num
     	FROM combine
    ) AS t1
  	WHERE topping_name IS NOT NULL
  	GROUP BY order_id, row_num
)

SELECT
	pizza_table.order_id,
	CONCAT(pizza_name,': ', ingredients) AS final_ingredients
FROM pizza_table
LEFT JOIN wording
USING(row_num)
````
#### Final Output
<img width="687" height="424" alt="image" src="https://github.com/user-attachments/assets/be4a42df-c759-4d96-8f24-5d08cee32156" />

## 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
#### SQL Query
````sql
WITH delivered AS (
	SELECT
  		order_id,
  		pizza_id,
  		exclusions,
  		extras
 	FROM customer_orders_clean
  	LEFT JOIN runner_orders_clean
  	USING(order_id)
  	WHERE pickup_time IS NOT NULL
),

exclusion AS (
  	SELECT
		toppings,
  		COUNT(*) * -1 AS amount
  	FROM (
    	SELECT
      		order_id,
  			REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS toppings
      	FROM delivered) AS t1
  	GROUP BY toppings
),

extra AS (
  	SELECT
		toppings,
  		COUNT(*) AS amount
  	FROM (
    	SELECT
      		order_id,
  			REGEXP_SPLIT_TO_TABLE(extras, ',') AS toppings
      	FROM delivered) AS t1
  	GROUP BY toppings	
),

regular AS (
	SELECT
  		toppings,
  		COUNT(*) AS amount
  	FROM (
      	SELECT
      		order_id,
  			REGEXP_SPLIT_TO_TABLE(toppings, ',') AS toppings
  		FROM delivered
  		LEFT JOIN pizza_runner.pizza_recipes
  		USING(pizza_id)) AS t1
  	GROUP BY toppings
),

total AS (
	SELECT *
  	FROM regular
  	UNION ALL
  	SELECT *
 	FROM exclusion
  	UNION ALL
  	SELECT *
  	FROM extra	
)

SELECT
	topping_name,
    SUM(amount) AS total_quantity
FROM total
LEFT JOIN pizza_runner.pizza_toppings AS t2
ON total.toppings :: INTEGER  = t2.topping_id
GROUP BY topping_name
ORDER BY total_quantity DESC;
````
#### Final Output
<img width="252" height="330" alt="image" src="https://github.com/user-attachments/assets/3496be7d-b837-490b-8904-1a0eea8f7160" />
