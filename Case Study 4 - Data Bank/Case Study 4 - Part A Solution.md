### 1. How many unique nodes are there on the Data Bank system?
````sql
SELECT
	COUNT(DISTINCT(region_id, node_id)) AS total_nodes
FROM data_bank.customer_nodes;
````

### 2. What is the number of nodes per region?
````sql
SELECT
	region_name,
    COUNT(DISTINCT(node_id)) AS nodes
FROM data_bank.customer_nodes
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_name;
````

### 3. How many customers are allocated to each region?
````sql
SELECT
	region_name,
    COUNT(DISTINCT(customer_id)) AS customers
FROM data_bank.customer_nodes
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_name;
````

### 4. How many days on average are customers reallocated to a different node?


### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

