Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```sql
SELECT	CASE 
			WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
			ELSE s.country
			END AS country,
		CASE 
			WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
			ELSE s.city
			END AS city,
		SUM(total_transaction_revenue::NUMERIC) AS transaction_revenue --COUNT(city)
FROM	
		all_sessions s
	/*Excluding the unknown cities and countries, this WHERE clause can be applied -- WHERE s.country not in ('(not set)', 'not available in demo dataset') 
			AND s.city not in ('(not set)', 'not available in demo dataset')*/
GROUP BY 
		country, city
HAVING 
		SUM(total_transaction_revenue::NUMERIC) IS NOT NULL
ORDER BY 
		transaction_revenue DESC;

--We can also use this query to assign nulls to unknown cities and countries.
/*SELECT    CASE 
			    WHEN country in ('(not set)', 'not available in demo dataset') THEN NULL
			    ELSE country
			    END AS country,
		    CASE 
                WHEN city in ('(not set)', 'not available in demo dataset') THEN NULL
                ELSE city
                END AS city,
                SUM(total_transaction_revenue::NUMERIC) AS transaction_revenue
FROM 		
		    all_sessions
WHERE 
		    country IS NOT NULL AND city IS NOT NULL
GROUP BY 
		    country, city
HAVING 
		    SUM(total_transaction_revenue::NUMERIC) IS NOT NULL --Add this as the query used to clean the data
ORDER BY 
		    transaction_revenue DESC;*/
```			



Answer: Country: United states have the highest transaction revenue. San Francisco is the city with the highest transaction revenue

![alt text](SQL_images/Question_1.png)


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

```sql
SELECT 	CASE 
				WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
				ELSE s.country
				END AS country,
		CASE 
				WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
				ELSE s.city
				END AS city,
				AVG(p.ordered_quantity) AS avg_quantity_ordered
FROM 	
		products p
JOIN 
		all_sessions s
ON 
		p.sku = s.sku
WHERE 
		s.country NOT IN ('(not set)', 'not available in demo dataset') 
		AND s.city NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY 
			country, city;


--alternatively

WITH avg_CTE AS (SELECT	CASE 
							WHEN s.country in ('(not set)', 'not available in demo dataset') THEN 'Unknown country'
							ELSE s.country
							END AS country,
						CASE 
							WHEN s.city in ('(not set)', 'not available in demo dataset') THEN 'Unknown city'
							ELSE s.city
							END AS city,
							s.sku
				FROM	all_sessions s
				WHERE 	s.country not in ('(not set)', 'not available in demo dataset') 
					    AND s.city not in ('(not set)', 'not available in demo dataset'))
SELECT 
		aq.country, 
		aq.city, 
		AVG(p.ordered_quantity) AS avg_quantity_ordered
FROM 
		avg_CTE aq
	JOIN 	
		products p
	ON 
		p.sku = aq.sku
GROUP BY 
		country, city;
```



Answer: This returned 268 rows with United States having the highest average ordered quantity

![alt text](SQL_images/Question_2.png)



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
WITH product_CAT_CTE AS (SELECT CASE 
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
SELECT  *
FROM    product_CAT_CTE pr
WHERE   pr.product_category != 'UNKNOWN' AND country NOT IN ('(not set)', 'not available in demo dataset')
```

Answer: Home/shop by/YouTube is ranked top by total order in most cities and countries

![alt text](SQL_images/Question_3.png)



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```sql
WITH top_selling_product AS (SELECT	
								CASE 
									WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
										ELSE s.country
									END AS country,
								CASE 
									WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
										ELSE s.city
									END AS city,
								p.product_name,
								SUM(a.units_sold::numeric) AS total_units_sold, 
								RANK() OVER (PARTITION BY country ORDER BY SUM(a.units_sold::numeric) DESC) AS Ranking
							FROM products p
								JOIN all_sessions s 
									ON p.sku = s.sku 
								JOIN analytics a 
									ON s.full_visitor_id = a.full_visitor_id 
						    GROUP BY 
								p.product_name, country, city
						    HAVING 
								SUM(a.units_sold::numeric) IS NOT NULL)
SELECT  *
FROM 
	    top_selling_product
```		

Answer: This returned 400 records
![alt text](SQL_images/Question_4.png)

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```sql
WITH total_revenue AS (SELECT	
							CASE 
								WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
									ELSE s.country
								END AS country,
							CASE 
								WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
									ELSE s.city
								END AS city,
							ROUND(SUM(units_sold::NUMERIC * unit_price::NUMERIC/1000000), 2) AS total_revenue, 
									RANK() OVER (PARTITION BY SUM(units_sold::NUMERIC * unit_price::NUMERIC/1000000) ORDER BY SUM(units_sold::NUMERIC * unit_price::NUMERIC/1000000) DESC) AS Ranking
					FROM all_sessions s
									JOIN analytics a 
									ON s.full_visitor_id = a.full_visitor_id 
					GROUP BY 
								country, city
					HAVING 
								SUM(total_transaction_revenue::NUMERIC) IS NOT NULL)
SELECT *
FROM 
		total_revenue;
		
		
--using the total_transaction_revenue data	
WITH total_revenue AS (SELECT	CASE 
									WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
									ELSE s.country
									END AS country,
								CASE 
									WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
									ELSE s.city
									END AS city,
								SUM(total_transaction_revenue::NUMERIC) AS total_revenue
						FROM all_sessions s
						GROUP BY 
								country, city
						HAVING 
								SUM(total_transaction_revenue::NUMERIC) IS NOT NULL)
SELECT *
FROM 
		total_revenue
ORDER BY total_revenue DESC;
```

Answer: United States is generating more revenues.

![alt text](SQL_images/Question_5.png)





