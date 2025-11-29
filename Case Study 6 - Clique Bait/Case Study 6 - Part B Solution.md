# Digital Analysis
## 1. How many users are there?
#### SQL Query
````sql
SELECT
	COUNT(DISTINCT(user_id)) AS total_users
FROM clique_bait.users;
````
#### Final Output
<img width="82" height="74" alt="image" src="https://github.com/user-attachments/assets/c9898456-1adb-47c0-b52b-d5e991159d13" />

## 2. How many cookies does each user have on average?
#### SQL Query
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
FROM user_cookies;
````
#### Final Output
<img width="112" height="74" alt="image" src="https://github.com/user-attachments/assets/74747011-315e-4a16-89f2-47abff26154e" />

## 3. What is the unique number of visits by all users per month?
#### SQL Query
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
#### Final Output
<img width="230" height="171" alt="image" src="https://github.com/user-attachments/assets/eb63aa33-2827-4489-8797-5a045602bef4" />

## 4. What is the number of events for each event type?
#### SQL Query
````sql
SELECT
	event_type,
    COUNT(*) AS num_of_events
FROM clique_bait.events
GROUP BY event_type
ORDER BY event_type;
````
#### Final Output
<img width="252" height="185" alt="image" src="https://github.com/user-attachments/assets/9b73154c-c044-4957-a676-73d8e159b220" />

## 5. What is the percentage of visits which have a purchase event?
#### SQL Query
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
#### Final Output
<img width="116" height="72" alt="image" src="https://github.com/user-attachments/assets/11bffe65-0c4e-4595-8783-bbdc21b54298" />

## 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
#### SQL Query
````sql
WITH conditional AS (
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
#### Final Output
<img width="111" height="78" alt="image" src="https://github.com/user-attachments/assets/aef91376-058a-4ded-9db4-c34d2dd7830b" />

## 7. What are the top 3 pages by number of views?
#### SQL Query
````sql
SELECT
	page_name,
    COUNT(*) AS number_of_views
FROM clique_bait.events
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
WHERE event_type = 1
GROUP BY page_name
ORDER BY number_of_views DESC
LIMIT 3;
````
#### Final Output
<img width="265" height="122" alt="image" src="https://github.com/user-attachments/assets/247009a5-79c5-48da-aab0-0e9745c81a22" />

## 8. What is the number of views and cart adds for each product category?
#### SQL Query
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
#### Final Output
<img width="439" height="121" alt="image" src="https://github.com/user-attachments/assets/2d8df9c1-6f8a-4d2b-afd2-e483a97d954c" />

## 9. What are the top 3 products by purchases?
#### SQL Query
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
#### Final Output
<img width="258" height="120" alt="image" src="https://github.com/user-attachments/assets/f977fa91-b588-4503-8197-5a3aa6c01b0a" />
