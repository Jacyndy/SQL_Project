Question 1: What is the variablility in transaction revenue by country

SQL Queries:
```sql
SELECT	CASE 
			WHEN country in ('(not set)', 'not available in demo dataset') THEN 'unknown'
			ELSE country
			END AS country,
			MAX(total_transaction_revenue::NUMERIC) AS Max_transaction_revenue,
			MIN(total_transaction_revenue::NUMERIC) AS Min_transaction_revenue --COUNT(city)
FROM	
		all_sessions
GROUP BY 
		country, transaction_revenue
HAVING 
		MAX(total_transaction_revenue::NUMERIC) IS NOT NULL
ORDER BY 
		Min_transaction_revenue DESC;
```
Answer: /*The minimium and maximium values in most countries are same. United States have the highest transaction revenue, 
followed by Israel.*/



Question 2: Determine if there are duplicates in the all_sessions table

SQL Queries:
```sql
SELECT      COUNT(full_visitor_id), full_visitor_id
FROM        all_sessions 
GROUP BY    full_visitor_id HAVING COUNT(full_visitor_id) > 1;
```

Answer: This returned 794 rows, hence showing duplicates in data



Question 3: What countries are are unique visitors in?


SQL Queries:
```sql
SELECT DISTINCT     country, COUNT(DISTINCT full_visitor_id) AS full_visitor_id
FROM                all_sessions
GROUP BY            country
ORDER BY            COUNT(DISTINCT full_visitor_id) DESC;
```

Answer: There are 136 countries, with UNited States having the highest number of visitors to the site.
