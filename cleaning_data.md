What issues will you address by cleaning the data?
--Data cleaning
When setting up the database, the columns were imported as text to avoid any data loss. The data types were then cast to the right data type before using them for analysis. Nulls were also excluded from the data.

Queries:
Below, provide the SQL queries you used to clean your data.

1. one of the data cleaning applied to the data used for this analysis was to first change the data type from text to 
character varying for the country and city column using the graphical interface. The total_transaction column was also 
cast to numeric. Transaction_revenue column was used for analysis because it contains more data than we have for total_transaction. In addition, the 4 records in the transaction_revenue column equals the records in the total_transaction_revenue column.

2. Another cleaning technique used in my analysis was assigning unknown city and country (in this data, it is referred to as 'not set' and 'not available in demo dataset', the name of the country and city respectively. These records were not excluded during the time of the analysis as we want to understand what revenue is genertated for each country and city. The query used to clean up the missing values is below:
A snapshot of the query used for question 1 in starting_with_question
```sql
SELECT	CASE 
			WHEN s.country in ('(not set)', 'not available in demo dataset') THEN city
			ELSE s.country
			END AS country,
		CASE 
			WHEN s.city in ('(not set)', 'not available in demo dataset') THEN country
			ELSE s.city
			END AS city...
```

3. The query below was also used to cast INT data type on ordered_quantity from the products table used for analysis.
ALTER TABLE products
ALTER COLUMN ordered_quantity TYPE INT
USING ordered_quantity::integer;

4. Nulls and missing data in the product category column from the all_sessions table used in the analysis was also fixed. Records stored as 'not set' and 'not available in demo dataset' were treated as unknown and filtered out from the analysis
A snapshot of the query used:
```SQL
SELECT	CASE 
			WHEN s.product_category_v2 in ('(not set)', 'not available in demo dataset') THEN 'UNKNOWN'
				ELSE s.product_category_v2
			END AS product_category, ....
```					

5. Units sold, unit price were cast as numeric for the analysis. This is done as the value for unit price is bigger than big int. The valus is also divided by 1000000 to get the value in millions. The snapshot of the query used is below:
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
						FROM all_sessions s...
```
6. Deleted duplicates of full_visitor_id from all_sessions table. To have unique full_visitor_id
```sql
DELETE FROM	all_sessions 
WHERE		full_visitor_id IN (
    		SELECT full_visitor_id FROM (
        	SELECT full_visitor_id, ROW_NUMBER() OVER (PARTITION BY full_visitor_id ORDER BY full_visitor_id) AS row_num
        	FROM all_sessions
    ) s WHERE s.row_num > 1
); 
/*deleted duplicates from the all_sessions table*/
```
