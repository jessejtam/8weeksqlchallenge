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


## 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"


## 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

