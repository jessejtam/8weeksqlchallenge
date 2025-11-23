### 1. How many unique nodes are there on the Data Bank system?
#### SQL Query
````sql
SELECT
	COUNT(DISTINCT(node_id)) AS unique_nodes
FROM data_bank.customer_nodes;
````
#### Final Output
<img width="98" height="71" alt="image" src="https://github.com/user-attachments/assets/3529e8c1-4b2a-41a2-b5da-b98be9b2b60d" />

### 2. What is the number of nodes per region?
#### SQL Query
````sql
SELECT
	region_name,
    COUNT(DISTINCT(node_id)) AS nodes
FROM data_bank.customer_nodes
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_name;
````
#### Final Output
<img width="239" height="174" alt="image" src="https://github.com/user-attachments/assets/1b34797c-d5f2-4b78-b179-d178e90fd40d" />

### 3. How many customers are allocated to each region?
#### SQL Query
````sql
SELECT
	region_name,
    COUNT(DISTINCT(customer_id)) AS customers
FROM data_bank.customer_nodes
LEFT JOIN data_bank.regions
USING(region_id)
GROUP BY region_name;
````
#### Final Output
<img width="244" height="177" alt="image" src="https://github.com/user-attachments/assets/c011d5f2-c99e-4055-9187-e3b1183628f9" />

### 4. How many days on average are customers reallocated to a different node?
#### SQL Query

#### Final Output

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
#### SQL Query

#### Final Output
