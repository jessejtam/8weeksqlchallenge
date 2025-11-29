# Campaign Analysis
## Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
#### SQL Query
````sql
CREATE TABLE records AS
SELECT
    user_id,
    t2.visit_id,
    MIN(t1.start_date) AS visit_start_time,
    SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_views,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
    SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase,
    campaign_name,
    SUM(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) AS impression,
    SUM(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) AS click,
    STRING_AGG(CASE WHEN event_type = 2 THEN page_name ELSE NULL END, ', ' ORDER BY sequence_number) AS cart_products
FROM clique_bait.users AS t1
LEFT JOIN clique_bait.events AS t2
USING(cookie_id)
LEFT JOIN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) AS purchased
ON t2.visit_id = purchased.visit_id
LEFT JOIN clique_bait.campaign_identifier AS t3
ON t1.start_date BETWEEN t3.start_date AND end_date
JOIN clique_bait.page_hierarchy
USING(page_id)
GROUP BY 1, 2, 7
ORDER BY 1, 3;
````
#### Final Output
<img width="1610" height="634" alt="image" src="https://github.com/user-attachments/assets/6816ae43-8733-478d-b954-a48eb1b61f3c" />
