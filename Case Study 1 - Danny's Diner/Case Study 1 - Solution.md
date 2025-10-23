##1. What is the total amount each customer spent at the restaurant?
````sql
SELECT s.customer_id, 
    SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m
USING(product_id)
GROUP BY s.customer_id
ORDER BY customer_id ASC;
````

##2. How many days has each customer visited the restaurant?
````sql
SELECT customer_id, 
    COUNT(DISTINCT(order_date)) AS visits
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC;
````

##3. What was the first item from the menu purchased by each customer?
````sql
WITH first_order AS (SELECT customer_id, 
        product_name,
        RANK() OVER(ORDER BY MIN(order_date)) AS order_rank
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    GROUP BY customer_id, product_name)

SELECT customer_id, 
    product_name
FROM first_order
WHERE order_rank = 1
ORDER BY customer_id;
````

##4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT product_name, 
    COUNT(*) AS times_purchased
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
USING(product_id)
GROUP BY product_name
ORDER BY times_purchased DESC
LIMIT 1;
````

##5. Which item was the most popular for each customer?
````sql
WITH popular_order AS (
    SELECT customer_id, 
        product_name,
        RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS order_rank
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    GROUP BY customer_id, product_name)
   
SELECT customer_id, 
    product_name
FROM popular_order
WHERE order_rank = 1;
````

##6. Which item was purchased first by the customer after they became a member?
````sql
WITH member_first_order AS (
    SELECT customer_id, 
        product_name,
        RANK() OVER(PARTITION BY customer_id ORDER BY MIN(order_date) ASC) AS date_rank
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.members
    USING(customer_id)
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    WHERE order_date >= join_date
    GROUP BY customer_id, product_name)
   
SELECT customer_id, 
    product_name
FROM member_first_order
WHERE date_rank = 1;
````

##7. Which item was purchased just before the customer became a member?
````sql
WITH nonmember_first_order AS (
    SELECT customer_id, 
        product_name,
        RANK() OVER(PARTITION BY customer_id ORDER BY MIN(order_date) DESC) AS date_rank
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.members
    USING(customer_id)
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    WHERE order_date < join_date
    GROUP BY customer_id, product_name)
   
SELECT customer_id, 
    product_name
FROM nonmember_first_order
WHERE date_rank = 1;
````

##8. What is the total items and amount spent for each member before they became a member?
````sql
WITH before_totals AS (
    SELECT customer_id, 
        COUNT(product_name) AS total_items, 
        SUM(price) AS total_spent
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.members
    USING(customer_id)
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    WHERE order_date < join_date
    GROUP BY customer_id
    ORDER BY customer_id)
   
SELECT customer_id, 
    total_items, 
    total_spent
FROM before_totals;
````

##9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
WITH points_table AS (
    SELECT customer_id,
        CASE WHEN product_name = 'sushi' THEN price * 20
            ELSE price * 10 END AS points
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.menu
    USING(product_id))
   
SELECT customer_id, 
    SUM(points) AS total_points
FROM points_table
GROUP BY customer_id
ORDER BY customer_id;
````

##10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
WITH points_table AS (
    SELECT customer_id,
        CASE WHEN order_date BETWEEN join_date AND join_date + 6 THEN price * 20
            WHEN product_name = 'sushi' THEN price * 20
            ELSE price * 10 END AS points
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    LEFT JOIN dannys_diner.members
    USING(customer_id)
    WHERE order_date <= '2021-01-31' AND 
        (customer_id = 'A' OR customer_id = 'B'))
   
SELECT customer_id, 
    SUM(points) AS total_points
FROM points_table
GROUP BY customer_id
ORDER BY customer_id;
````

##BONUS QUESTION 1: JOIN ALL THE THINGS
````sql
SELECT customer_id, 
    order_date, 
    product_name, 
    price,
    CASE WHEN order_date >= join_date THEN 'Y'
        ELSE 'N' END AS member
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
USING(product_id)
LEFT JOIN dannys_diner.members
USING(customer_id)
ORDER BY customer_id, order_date, product_name
````

##BONUS QUESTION 2: RANK ALL THE THINGS
````sql
WITH member_table AS (
    SELECT customer_id, 
        order_date, 
        product_name, 
        price,
        CASE WHEN order_date >= join_date THEN 'Y'
            ELSE 'N' END AS member
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.menu
    USING(product_id)
    LEFT JOIN dannys_diner.members
    USING(customer_id))

SELECT *,
    CASE WHEN member = 'N' THEN null
        ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date ASC) END AS ranking 
FROM member_table
ORDER BY customer_id ASC, order_date ASC, product_name ASC;
````
