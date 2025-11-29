## Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
#### SQL Query
````sql
CREATE TABLE product_output AS
SELECT
    product_id,
    page_name AS product,
    SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS total_views,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS total_to_carts,
    SUM(CASE WHEN event_type = 2 AND purchased.visit_id IS NULL THEN 1 ELSE 0 END) AS total_abondoned,
    SUM(CASE WHEN event_type = 2 AND purchased.visit_id IS NOT NULL THEN 1 ELSE 0 END) AS total_purchased
FROM clique_bait.events AS t1
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
LEFT JOIN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) AS purchased
ON t1.visit_id = purchased.visit_id
WHERE product_id IS NOT NULL
GROUP BY product_id, product
ORDER BY product_id;
````
#### Final Output
<img width="859" height="289" alt="image" src="https://github.com/user-attachments/assets/95ea5099-68da-438b-98ad-4da0a3fcfec2" />

## Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
#### SQL Query
````sql
CREATE TABLE category_output AS
SELECT
    product_category,
    SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS total_views,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS total_to_carts,
    SUM(CASE WHEN event_type = 2 AND purchased.visit_id IS NULL THEN 1 ELSE 0 END) AS total_abondoned,
    SUM(CASE WHEN event_type = 2 AND purchased.visit_id IS NOT NULL THEN 1 ELSE 0 END) AS total_purchased
FROM clique_bait.events AS e
LEFT JOIN clique_bait.page_hierarchy
USING(page_id)
LEFT JOIN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) AS purchased
ON e.visit_id = purchased.visit_id
WHERE product_id IS NOT NULL
GROUP BY product_category
ORDER BY product_category;
````
#### Final Output
<img width="711" height="128" alt="image" src="https://github.com/user-attachments/assets/9249d740-7587-4360-aa3f-6cc2727e2e04" />

## Use your 2 new output tables - answer the following questions:
## 1. Which product had the most views, cart adds and purchases?
#### SQL Query
````sql
SELECT
    (SELECT product FROM product_output ORDER BY total_views DESC LIMIT 1) AS most_views,
    (SELECT product FROM product_output ORDER BY total_to_carts DESC LIMIT 1) AS most_to_carts,
    (SELECT product FROM product_output ORDER BY total_purchased DESC LIMIT 1) AS most_purchased
````
#### Final Output
<img width="410" height="74" alt="image" src="https://github.com/user-attachments/assets/a340c9af-0cdb-49cc-a170-816c79436772" />

## 2. Which product was most likely to be abandoned?
#### SQL Query
````sql
SELECT
    product AS most_likely_abondoned
FROM product_output
ORDER BY total_abondoned DESC
LIMIT 1;
````
#### Final Output
<img width="154" height="77" alt="image" src="https://github.com/user-attachments/assets/ec82c562-cdad-4ed4-a763-388e6310d1ac" />

## 3. Which product had the highest view to purchase percentage?
#### SQL Query
````sql
SELECT
    product,
    ROUND(100.0 * total_purchased / total_views, 2) AS view_to_purchase_percent
FROM product_output
ORDER BY view_to_purchase_percent DESC
LIMIT 1;
````
#### Final Output
<img width="321" height="76" alt="image" src="https://github.com/user-attachments/assets/15a498a2-ea28-41f6-a25a-520febfca1c2" />

## 4. What is the average conversion rate from view to cart add?
#### SQL Query
````sql
SELECT
    ROUND(AVG(100.0 * total_to_carts / total_views), 2) AS avg_view_to_cart_percent
FROM product_output;
````
#### Final Output
<img width="170" height="75" alt="image" src="https://github.com/user-attachments/assets/57be3a79-6c09-4936-83dc-53bb7b019cb2" />

## 5. What is the average conversion rate from cart add to purchase?
#### SQL Query
````sql
SELECT
    ROUND(AVG(100.0 * total_purchased / total_to_carts), 2) AS avg_cart_to_purchased_percent
FROM product_output;
````
#### Final Output
<img width="209" height="73" alt="image" src="https://github.com/user-attachments/assets/8df0e75f-4fae-4b89-ab14-f75f46c8da56" />
