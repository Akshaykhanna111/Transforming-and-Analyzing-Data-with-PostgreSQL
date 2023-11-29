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
```sql

DROP TABLE units_city_country CASCADE;
create temp table units_city_country AS
	(SELECT T3.UNIQUE_ID,
	 		T3.total_units,
			T4.CITY,
			T4.COUNTRY
		FROM
			(select UNIQUE_ID, 
			 SUM(units_sold) as total_units
			 from 
			 (SELECT CONCAT(FULL_VISITOR_ID,
					VISIT_ID) AS UNIQUE_ID,
			 		REVENUE,
					units_sold AS units_sold
				FROM PUBLIC.REVENUE_ANALYTICS_TABLE) t1
				GROUP BY UNIQUE_ID
			 
				UNION 
			 	select UNIQUE_ID, 
				 SUM(total_units_sold) as total_units
				 from 
				 (SELECT DISTINCT 
					CONCAT(FULL_VISITOR_ID,
					VISIT_ID) UNIQUE_ID,
					total_transaction_revenue,
				  	product_quantity AS total_units_sold
				  FROM PUBLIC.CLEANED_SESSION_DETAILS
					WHERE TRANSACTION_ID like '%ORD%'
						OR TRANSACTIONS = '1') t2
			 GROUP BY UNIQUE_ID) T3
			JOIN
			(SELECT DISTINCT CONCAT(FULL_VISITOR_ID,
					VISIT_ID) UNIQUE_ID,
					COUNTRY,
					CITY
				FROM PUBLIC.CLEANED_SESSION_DETAILS) T4 ON 
	 			T3.UNIQUE_ID = T4.UNIQUE_ID) 



select city, country, avg(total_units) as avg_qty_sold 
from units_city_country
group by city, country
having avg(total_units) > 0
order by avg_qty_sold desc

```
Owing to very less matches between sessions and analytics tables plus the filter of transactionid being applied on sessions - total count of rows is 101 and hence the avg qty will be on the lower side due to lack of sufficient data

Answer:
Following are the top 5 city and country combinations (after excluding scenarios where junk values are present in city and country)-  
City | Country | Avg Qty  
Sunnyvale | United States | 15  
Atlanta	| United States	| 4  
San Francisco | United States | 3  
Seattle | United States | 2  





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

```sql
-- For this question we can only explore the sessions table
-- as a unique id combination might be mapped to multiple 
-- categories and hence there's no way to pick units sold from 
-- the analytics table. 

create temp table units_city_country_category AS
select city, country,product_category,
sum(coalesce(product_quantity, 0)) as total_units_sold_per_group,
(select sum(coalesce(product_quantity, 0)) from 
PUBLIC.CLEANED_SESSION_DETAILS) as total_units_sold
from 
PUBLIC.CLEANED_SESSION_DETAILS
WHERE TRANSACTION_ID like '%ORD%'
OR TRANSACTIONS = '1'
group by city, country,product_category
having sum(coalesce(product_quantity, 0)) > 0
order by 4 desc

-- At a city country level
select city,country, product_category,
sum(total_units_sold_per_group) as total_units
from units_city_country_category
group by city,country, product_category
order by 4 desc

-- At a city level
select city,product_category,
sum(total_units_sold_per_group) as total_units
from units_city_country_category
group by city,product_category
order by 3 desc 
-- for more than 50% of data, the city is not available
-- Atlanta, Palo Alto and Mountain View are the top 3 cities
-- with Bags and Nest-USA as the preferred categories

-- At a country level
select country,product_category,
sum(total_units_sold_per_group) as total_units
from units_city_country_category
group by country,product_category
order by 3 desc 
-- Since US contributes upto 93% of order volume, 
-- most of the orders are from US. Within US the preference
-- is for following 2 categories - 
-- 1. Home/Office/Notebooks & Journals/ - (34%)
-- 2. Bags - 29%

```


Answer:

At a country level - 
Since US contributes upto 93% of order volume, most of the orders are from US. Within US the preference is for following 2 categories - 
1. Home/Office/Notebooks & Journals/ - (34%)
2. Bags - 29%

At a city level
For more than 50% of data, the city is not available. 
Atlanta, Palo Alto and Mountain View are the top 3 cities with Bags and Nest-USA as the preferred categories

Further analysis can be done to look for seasonal patterns but owing to data volume very low (81 transactions with successful order id) it won't be feasible in the current scope. 

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







