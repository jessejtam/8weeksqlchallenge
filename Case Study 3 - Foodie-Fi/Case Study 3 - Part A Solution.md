## Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
## Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
#### SQL Query
````sql
SELECT
	customer_id,
    plan_name,
    start_date
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING(plan_id)
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY customer_id ASC, start_date ASC;
````
#### Final Output
<img width="384" height="588" alt="image" src="https://github.com/user-attachments/assets/b77c0a50-d236-42cd-8c2b-4417a16ad4de" />
