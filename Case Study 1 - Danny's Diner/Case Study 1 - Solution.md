# Case Study Questions
## 1. What is the total amount each customer spent at the restaurant?
#### SQL Query
````sql
SELECT s.customer_id, 
    SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m
USING(product_id)
GROUP BY s.customer_id
ORDER BY customer_id ASC;
````
#### Final Output
<img width="235" height="133" alt="image" src="https://github.com/user-attachments/assets/80d30851-3e44-4644-b44f-f88c6f4d2615" />

## 2. How many days has each customer visited the restaurant?
#### SQL Query
````sql
SELECT customer_id, 
    COUNT(DISTINCT(order_date)) AS visits
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC;
````
#### Final Output
<img width="232" height="127" alt="image" src="https://github.com/user-attachments/assets/48e55e63-5079-4203-a7e1-01265126fcf7" />

## 3. What was the first item from the menu purchased by each customer?
#### SQL Query
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
#### Final Output
<img width="256" height="142" alt="image" src="https://github.com/user-attachments/assets/c3ea6732-cc6a-4fff-9915-17390090287b" />

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
#### SQL Query
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
#### Final Output
<img width="267" height="75" alt="image" src="https://github.com/user-attachments/assets/b9f845e5-ec22-4b62-8f9a-54fb0cca2ddb" />

## 5. Which item was the most popular for each customer?
#### SQL Query
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
#### Final Output
<img width="260" height="171" alt="image" src="https://github.com/user-attachments/assets/a329b0b2-5b20-4f3c-9308-8e3619a90b6b" />

## 6. Which item was purchased first by the customer after they became a member?
#### SQL Query
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
#### Final Output
<img width="255" height="97" alt="image" src="https://github.com/user-attachments/assets/b3722d51-a9f3-46e4-9a14-88721f1a168b" />

## 7. Which item was purchased just before the customer became a member?
#### SQL Query
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
#### Final Output
<img width="250" height="123" alt="image" src="https://github.com/user-attachments/assets/6fc8b94f-85dd-4fc4-897e-37597c9ef744" />

## 8. What is the total items and amount spent for each member before they became a member?
#### SQL Query
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
#### Final Output
<img width="389" height="106" alt="image" src="https://github.com/user-attachments/assets/b5358f1d-2c12-4fa2-902a-7cc9210e41f4" />

## 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
#### SQL Query
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
#### Final Output
<img width="244" height="125" alt="image" src="https://github.com/user-attachments/assets/da025e7e-d84b-45cd-bb12-e8a142f4e9fb" />

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
#### SQL Query
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
#### Final Output
<img width="245" height="97" alt="image" src="https://github.com/user-attachments/assets/62e53bd6-8b0b-4b39-b6a8-73c69fb1d822" />

# Bonus Questions
## Bonus Question 1: Join All The Things
#### SQL Query
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
#### Final Output
<img width="685" height="454" alt="image" src="https://github.com/user-attachments/assets/b0fc47e7-2fdc-41e7-907d-572e7af0cc15" />

## BONUS QUESTION 2: RANK ALL THE THINGS
#### SQL Query
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
#### Final Output
<img width="872" height="462" alt="image" src="https://github.com/user-attachments/assets/2efc50f8-68f8-4daf-bf98-1586cca254ef" />
