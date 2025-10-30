## 1. What are the standard ingredients for each pizza?
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

## 2. What was the most commonly added extra?
````sql
WITH toppings AS (
	SELECT
  		REGEXP_SPLIT_TO_TABLE(extras, ',') AS extra
  	FROM pizza_runner.customer_orders
)

SELECT
	topping_name,
    COUNT(extra) AS times_added
FROM toppings AS t1
LEFT JOIN pizza_runner.pizza_toppings AS t2
ON t1.extra :: INTEGER = t2.topping_id
GROUP BY topping_name
ORDER BY times_added DESC;
````

## 3. What was the most common exclusion?
````sql
WITH toppings AS (
	SELECT
  		REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS exclusion
  	FROM pizza_runner.customer_orders
)

SELECT
	topping_name,
    COUNT(exclusion) AS times_excluded
FROM toppings AS t1
LEFT JOIN pizza_runner.pizza_toppings AS t2
ON t1.exclusion :: INTEGER = t2.topping_id
GROUP BY topping_name
ORDER BY times_excluded DESC;
````

## 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
### - Meat Lovers
### - Meat Lovers - Exclude Beef
### - Meat Lovers - Extra Bacon
### - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
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
  		FROM pizza_runner.customer_orders) AS t1 
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
  		FROM pizza_runner.customer_orders) AS t1 
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
  	FROM pizza_runner.customer_orders
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

## 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"


## 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
WITH exclusion AS (
  	SELECT
  		order_id,
      	ROW_NUMBER() OVER(ORDER BY order_id ASC, pizza_id ASC, exclusions DESC) AS rn,
  		REGEXP_SPLIT_TO_TABLE(exclusions, ',') AS topping
  	FROM pizza_runner.customer_orders
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
  		FROM pizza_runner.customer_orders) AS t1 
  	LEFT JOIN pizza_runner.pizza_toppings AS t2
  	ON t1.topping :: INTEGER = t2.topping_id
  	GROUP BY order_id, rn	
),

regular AS (
	SELECT
  		topping_name,
  		COUNT(*) AS amount
  	FROM (
      	SELECT
  			order_id,
  			REGEXP_SPLIT_TO_TABLE(toppings, ',') AS toppings
  		FROM pizza_runner.customer_orders
  		LEFT JOIN pizza_runner.pizza_recipes
  		USING(pizza_id)) AS t1
  	LEFT JOIN pizza_runner.pizza_toppings AS t2
  	ON t1.toppings :: INTEGER = t2.topping_id
  	GROUP BY topping_name
) 

SELECT *
FROM regular
