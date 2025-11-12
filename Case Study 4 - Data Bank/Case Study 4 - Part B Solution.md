### 1. What is the unique count and total amount for each transaction type?
````sql
SELECT
	txn_type,
    COUNT(*) AS num_of_transactions,
    SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
````

### 2. What is the average total historical deposit counts and amounts for all customers?
````sql
WITH count_table AS (
	SELECT
  		customer_id,
  		COUNT(*) AS deposit_count,
  		SUM(txn_amount) AS deposit_total
  	FROM data_bank.customer_transactions
  	WHERE txn_type = 'deposit'
  	GROUP BY customer_id
)

SELECT
	ROUND(AVG(deposit_count), 3) AS avg_deposit_count,
    ROUND(AVG(deposit_total), 2) AS avg_deposit_amount
FROM count_table;
````

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
````sql
WITH deposit_table AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		COUNT(*) AS deposit_count
  	FROM data_bank.customer_transactions
  	WHERE txn_type = 'deposit'
  	GROUP BY customer_id, month
),

other_table AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		COUNT(*) AS other_count
  	FROM data_bank.customer_transactions
  	WHERE txn_type <> 'deposit'
  	GROUP BY customer_id, month
)

SELECT
	t1.month,
    COUNT(*) AS customers
FROM deposit_table AS t1
LEFT JOIN other_table AS t2
ON t1.customer_id = t2.customer_id AND t1.month = t2.month
WHERE deposit_count > 1 AND other_count > 1
GROUP BY t1.month
ORDER BY
	CASE WHEN t1.month = 'January' THEN 1
    WHEN t1.month = 'February' THEN 2
    WHEN t1.month = 'March' THEN 3
    WHEN t1.month = 'April' THEN 4
END;
````

### 4. What is the closing balance for each customer at the end of the month?
````sql
WITH deposits AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		txn_amount
  	FROM data_bank.customer_transactions
  	WHERE txn_type = 'deposit'
),

other AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		txn_amount * -1
  	FROM data_bank.customer_transactions
  	WHERE txn_type <> 'deposit'
),

combine AS (
	SELECT *
  	FROM deposits
  	UNION
  	SELECT *
  	FROM other
)

SELECT
	customer_id,
	month,
    SUM(txn_amount) AS closing_balance
FROM combine
GROUP BY customer_id, month
ORDER BY customer_id,
	CASE WHEN month = 'January' THEN 1
    	WHEN month = 'February' THEN 2
    	WHEN month = 'March' THEN 3
    	WHEN month = 'April' THEN 4
	END;
````

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
````sql
WITH deposits AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		txn_amount
  	FROM data_bank.customer_transactions
  	WHERE txn_type = 'deposit'
),

other AS (
	SELECT
  		customer_id,
  		TO_CHAR(txn_date, 'FMMonth') AS month,
  		txn_amount * -1
  	FROM data_bank.customer_transactions
  	WHERE txn_type <> 'deposit'
),

combine AS (
	SELECT *
  	FROM deposits
  	UNION
  	SELECT *
  	FROM other
),

closing AS (
	SELECT
		customer_id,
		month,
    	SUM(txn_amount) AS closing_balance
	FROM combine
	GROUP BY customer_id, month
),

ranked AS (
	SELECT
		customer_id,
		month,
    	closing_balance,
    	LEAD(closing_balance) OVER(PARTITION BY customer_id ORDER BY
        	CASE WHEN month = 'January' THEN 1
    			WHEN month = 'February' THEN 2
    			WHEN month = 'March' THEN 3
    			WHEN month = 'April' THEN 4
        	END) AS next_balance,
  		RANK() OVER(PARTITION BY customer_id ORDER BY
        	CASE WHEN month = 'January' THEN 1
    			WHEN month = 'February' THEN 2
    			WHEN month = 'March' THEN 3
    			WHEN month = 'April' THEN 4
            END) AS month_rank
	FROM closing
),

initial_balance AS (
	SELECT
  		customer_id,
  		closing_balance AS initial_bal
  	FROM ranked
  	WHERE month_rank = 1
),

final_balance AS (
	SELECT
  		customer_id,
  		closing_balance AS final_bal
  	FROM ranked
  	WHERE next_balance IS NULL
)

SELECT
	100 * COUNT(*) :: DECIMAL / 500 AS percentage
FROM initial_balance
LEFT JOIN final_balance
USING(customer_id)
WHERE final_bal > 1.05 * initial_bal AND final_bal IS NOT NULL AND initial_bal IS NOT NULL;
````
