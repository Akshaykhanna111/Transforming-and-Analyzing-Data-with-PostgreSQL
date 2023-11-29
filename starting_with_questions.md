Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

```sql
-- To answer this question first we will check the 
-- unique combination of city, country, fullvisitorID and visitID

WITH CTE_CITY_COUNTRY AS
	(SELECT *,
			ROW_NUMBER() OVER(PARTITION BY UNIQUE_ID) AS ROW_NUMBERS
		FROM
			(SELECT DISTINCT CONCAT(FULL_VISITOR_ID,																					VISIT_ID) UNIQUE_ID,
					COUNTRY,
					CITY
				FROM PUBLIC.CLEANED_SESSION_DETAILS))
SELECT *
FROM CTE_CITY_COUNTRY
WHERE UNIQUE_ID IN
		(SELECT UNIQUE_ID
			FROM CTE_CITY_COUNTRY
			WHERE ROW_NUMBERS > 1)
ORDER BY UNIQUE_ID
-- This means that the combination is unique for a city and country

-- Now to get the revenue there are 2 tables - 
-- public.revenue_analytics_table analytics 
-- public.cleaned_session_details

-- We will do a union of total revenue from both
-- Also note, that we are assuming that in each visit number 
-- (in analytics table) if revenue is captured then it's a unique
-- transaction and not repeating across visits for a customer. 

-- Query to get the proportion by combination
WITH CTE_COUNTRY_CITY_REVENUE AS
	(SELECT T1.*,
			T2.CITY,
			T2.COUNTRY
		FROM
			(SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) AS UNIQUE_ID,
					SUM(REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.REVENUE_ANALYTICS_TABLE
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)
				UNION SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) UNIQUE_ID,
					SUM(TOTAL_TRANSACTION_REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.CLEANED_SESSION_DETAILS
				WHERE TRANSACTION_ID like '%ORD%'
					OR TRANSACTIONS = '1'
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)) T1
		JOIN
			(SELECT DISTINCT CONCAT(FULL_VISITOR_ID,
					VISIT_ID) UNIQUE_ID,
					COUNTRY,
					CITY
				FROM PUBLIC.CLEANED_SESSION_DETAILS) T2 ON T1.UNIQUE_ID = T2.UNIQUE_ID) -- here we do not need condition
-- for transactions and transactionID as we have checked
-- that totalRevneue is only present for scenarios
-- where either transaction flag is 1 or
-- transactionID is mapped. Still it's been added to avoid
-- any discrepancy due to data related issues)



SELECT COUNTRY,
	CITY,
	SUM(TOTAL_REVENUE), (select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE),
(SUM(TOTAL_REVENUE)/(select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE))*100		
FROM CTE_COUNTRY_CITY_REVENUE
GROUP BY COUNTRY,
	CITY
ORDER BY SUM(TOTAL_REVENUE) DESC


-- Query to get the proportion by country
WITH CTE_COUNTRY_CITY_REVENUE AS
	(SELECT T1.*,
			T2.CITY,
			T2.COUNTRY
		FROM
			(SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) AS UNIQUE_ID,
					SUM(REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.REVENUE_ANALYTICS_TABLE
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)
				UNION SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) UNIQUE_ID,
					SUM(TOTAL_TRANSACTION_REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.CLEANED_SESSION_DETAILS
				WHERE TRANSACTION_ID like '%ORD%'
					OR TRANSACTIONS = '1'
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)) T1
		JOIN
			(SELECT DISTINCT CONCAT(FULL_VISITOR_ID,
					VISIT_ID) UNIQUE_ID,
					COUNTRY,
					CITY
				FROM PUBLIC.CLEANED_SESSION_DETAILS) T2 ON T1.UNIQUE_ID = T2.UNIQUE_ID) -- here we do not need condition
-- for transactions and transactionID as we have checked
-- that totalRevneue is only present for scenarios
-- where either transaction flag is 1 or
-- transactionID is mapped. Still it's been added to avoid
-- any discrepancy due to data related issues)



SELECT COUNTRY,
--	CITY,
	SUM(TOTAL_REVENUE), (select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE),
(SUM(TOTAL_REVENUE)/(select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE))*100		
FROM CTE_COUNTRY_CITY_REVENUE
GROUP BY COUNTRY--,CITY
ORDER BY SUM(TOTAL_REVENUE) DESC


-- Query to get the proportion by city
WITH CTE_COUNTRY_CITY_REVENUE AS
	(SELECT T1.*,
			T2.CITY,
			T2.COUNTRY
		FROM
			(SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) AS UNIQUE_ID,
					SUM(REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.REVENUE_ANALYTICS_TABLE
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)
				UNION SELECT DISTINCT FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID) UNIQUE_ID,
					SUM(TOTAL_TRANSACTION_REVENUE) AS TOTAL_REVENUE
				FROM PUBLIC.CLEANED_SESSION_DETAILS
				WHERE TRANSACTION_ID like '%ORD%'
					OR TRANSACTIONS = '1'
				GROUP BY FULL_VISITOR_ID,
					VISIT_ID,
					CONCAT(FULL_VISITOR_ID,
						VISIT_ID)) T1
		JOIN
			(SELECT DISTINCT CONCAT(FULL_VISITOR_ID,
					VISIT_ID) UNIQUE_ID,
					COUNTRY,
					CITY
				FROM PUBLIC.CLEANED_SESSION_DETAILS) T2 ON T1.UNIQUE_ID = T2.UNIQUE_ID) -- here we do not need condition
-- for transactions and transactionID as we have checked
-- that totalRevneue is only present for scenarios
-- where either transaction flag is 1 or
-- transactionID is mapped. Still it's been added to avoid
-- any discrepancy due to data related issues)



SELECT --COUNTRY,
CITY,
	SUM(TOTAL_REVENUE), (select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE),
(SUM(TOTAL_REVENUE)/(select SUM(TOTAL_REVENUE) FROM
						CTE_COUNTRY_CITY_REVENUE))*100		
FROM CTE_COUNTRY_CITY_REVENUE
GROUP BY CITY --COUNTRY
ORDER BY SUM(TOTAL_REVENUE) DESC
```
Answer:
Top 3 Countries - 
1. USA ~ 93%
2. Israel ~ 4%
3. Australia ~ 2.4%

Top 3 Cities (after excluding records where city was not available)-
1. Sunnyvale ~ 11%
2. San Francisco ~ 11%
3. Atlanta ~ 6%


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







