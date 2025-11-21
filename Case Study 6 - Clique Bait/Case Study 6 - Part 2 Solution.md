### 1. How many users are there?
````sql
SELECT
	COUNT(DISTINCT(user_id)) AS total_users
FROM clique_bait.users;
````

### 2. How many cookies does each user have on average?
````sql
WITH user_cookies AS (
	SELECT
  		user_id,
  		COUNT(cookie_id) AS total_cookies
  	FROM clique_bait.users
  	GROUP BY user_id
)

SELECT
	ROUND(AVG(total_cookies), 3) AS average_cookies
FROM user_cookies
````

### 3. What is the unique number of visits by all users per month?
````sql
WITH months AS (
	SELECT
  		TO_CHAR(event_time, 'FMMonth') AS month,
  		COUNT(DISTINCT(visit_id)) AS visits
  	FROM clique_bait.events
  	GROUP BY month
)

SELECT
	month,
    visits
FROM months
ORDER BY
	CASE WHEN month = 'January' THEN 1
    	WHEN month = 'February' THEN 2
        WHEN month = 'March' THEN 3
        WHEN month = 'April' THEN 4
        WHEN month = 'May' THEN 5
    END;
````

### 4. What is the number of events for each event type?
````sql
SELECT
	event_type,
    COUNT(*) AS num_of_events
FROM clique_bait.events
GROUP BY event_type
ORDER BY event_type;
````

### 5. What is the percentage of visits which have a purchase event?
````sql
WITH purchased AS (
	SELECT
  		COUNT(DISTINCT(visit_id)) AS purchase_count
  	FROM clique_bait.events
  	LEFT JOIN clique_bait.event_identifier
  	USING(event_type)
  	WHERE event_name = 'Purchase'
),

total AS (
	SELECT
  		COUNT(DISTINCT(visit_id)) AS total_count
  	FROM clique_bait.events
)

SELECT
	ROUND(100.0 * purchase_count / total_count, 2) AS purchase_percent
FROM purchased
CROSS JOIN total;
````

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
````sql
conditional AS (
	SELECT
  		COUNT(DISTINCT(visit_id)) AS condition_count
  	FROM clique_bait.events
  	WHERE visit_id NOT IN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) AND page_id = 12
),

total AS (
	SELECT
  		COUNT(DISTINCT(visit_id)) AS total_count
  	FROM clique_bait.events
)

SELECT
	ROUND(100.0 * condition_count / total_count, 2) AS percent_of_visits
FROM conditional
CROSS JOIN total;
````

### 7. What are the top 3 pages by number of views?
````sql
SELECT
	page_name,
    COUNT(*) AS number_of_views
FROM clique_bait.events
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
WHERE event_type = 1
GROUP BY page_name
ORDER BY number_of_visits DESC
LIMIT 3;
````

### 8. What is the number of views and cart adds for each product category?
````sql
SELECT
	product_category,
    SUM(CASE WHEN event_type = 1 THEN 1 END) AS number_of_views,
    SUM(CASE WHEN event_type = 2 THEN 1 END) AS number_of_cartadds
FROM clique_bait.events
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
WHERE product_category IS NOT NULL
GROUP BY product_category;
````

### 9. What are the top 3 products by purchases?
````sql
SELECT
	page_name,
    COUNT(*) AS total_purchased
FROM clique_bait.events
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
WHERE event_type = 2 AND visit_id IN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3)
GROUP BY page_name
ORDER BY total_purchased DESC
LIMIT 3;
````
