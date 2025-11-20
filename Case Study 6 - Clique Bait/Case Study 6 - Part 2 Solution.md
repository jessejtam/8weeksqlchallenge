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


### 4. What is the number of events for each event type?


### 5. What is the percentage of visits which have a purchase event?


### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?


### 7. What are the top 3 pages by number of views?


### 8. What is the number of views and cart adds for each product category?


### 9. What are the top 3 products by purchases?

