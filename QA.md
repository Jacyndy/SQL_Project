What are your risk areas? Identify and describe them.



QA Process:
Describe your QA process and include the SQL queries used to execute it.
1. To check if we are returning the required number of cities used in analysis
```sql
SELECT      country, 
		    COUNT(city) 
FROM 	    all_sessions 
GROUP BY    country, city
HAVING      SUM(total_transaction_revenue::NUMERIC) IS NOT NULL
ORDER BY    country; 
/*the total number of cities returned from the count function is 8,279. Which is same as the total number of cities 
returned if the count function is included in the query for question 1 of starting with questions*/
```
2. -- used to determine the countries available
```sql
SELECT      country, 
		    COUNT(country) 
FROM 	    all_sessions 
GROUP BY    city 
ORDER BY    city; 
```
3. /*This query was used to check for the total number records returned with relation to the query used in question 2. This gives one of the risk areas, where we have different records returned between the two queries.*/
```sql
SELECT      sku, city
FROM        all_sessions 
WHERE       country NOT IN ('(not set)', 'not available in demo dataset')  
            AND city not in ('(not set)', 'not available in demo dataset')
GROUP BY    sku, city;
```
4. QA query for question 3
```sql
WITH product_CAT_CTE AS (SELECT		CASE 
					WHEN s.product_category_v2 in ('(not set)', 'not available in demo dataset') THEN 'UNKNOWN'
					ELSE s.product_category_v2
					END AS product_category,
			CASE 
					WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
					ELSE s.country
					END AS country,
			CASE 
					WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
					ELSE s.city
					END AS city,
			SUM(p.ordered_quantity), 
			RANK() OVER (PARTITION BY country ORDER BY SUM(p.ordered_quantity) DESC) AS Ranking
		FROM all_sessions s
			JOIN 
			products p
			ON 
			s.sku = p.sku
			GROUP BY product_category_v2, country, city)
SELECT      *, 
            COUNT(product_category)
FROM        product_CAT_CTE pr
WHERE       pr.product_category != 'UNKNOWN' 
            AND country NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY    product_category, 
            country, 
            city, 
            sum, 
            ranking;

SELECT      COUNT(DISTINCT product_category_v2)
FROM        all_sessions
GROUP BY    country, city
```





