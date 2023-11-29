What issues will you address by cleaning the data?

In cleaning code, we will be checking for follwing issues - 
1. Data type mismatch
2. Null or junk values in categorical values
3. Anomalies in the dataset
4. Referential integrity to some extent



Queries:
Below, provide the SQL queries you used to clean your data.


```sql
-- The first steps after data upload are validation of count of records, 
-- checking sum of numerical columns, looking for corruption of data etc. Post that create a schema design of 
-- normalized database. 

-- In the current files the data is not normalized and lots of 
-- redundant fields are present across files. Also in some files 
-- duplicates are present whereas in few there are lot's of non 
-- prime attributes that are not dependent on the prime attributes or
-- partially dependent on other non prime attributes. 

-- Post reviewing the data structure following steps have been taken
-- to create a normalized table schema

-- Files -> Products, Sales_Report and Sales by SKU
-- All the attributes in these files are dependent on the SKU
-- since all the fields are pertaining to SKUs only. Thus rather than 
-- distributing this data across 3 files we can club all the attributes
-- in a single table consisting of SKU.
-- Also, Sales report seems to be a report created after joining products
-- table with sales_by_sku. So we can ignore it and create the merged
-- table using products 

-- Thus, we will be creating one Product table for product
-- related attributes. Below is the create table query. 

-- Also, while creating this table, one more very important
-- step would be to check for referential integrity. Since SKU
-- will be a foreign key in the sessions table, we have to ensure tha
-- SKUs that are present in sessions table, should be a part of
-- products table. On analyzing the data, it was found that 2033 
-- records were present in the SKU column in all_sessions table
-- that are not present in the products table. So we have to include 
-- those as well at the time of creation of products table. 

-- Ideally the table structures should be like this - 

	-- Product_Table - Product ID, Variant ID, Supplier ID, Price, Brand, Stock 
	-- restocking lead time, sentiment score, sentiment magnitude etc. 

	-- Product_Category - Product ID and Category ID

	-- Category_Table - Category ID, Details about the category

	-- Variant Table - Variant ID, Variant Details

	-- Supplier Table - Supplier ID, supplier details

	-- Product Order Table - Product ID, Order ID

	-- Order Table - Order ID, Customer ID, Date of order, etc. 

	-- Customer Table - Customer ID, customer demographic details 


-- DROP TABLE public.product_details CASCADE;

CREATE TABLE public.product_details
(
    "SKU" character varying NOT NULL,
    name character varying,
    "orderedQuantity" integer,
    "stockLevel" integer,
    "restockingLeadTime" integer,
    "sentimentScore" real,
    "sentimentMagnitude" real,
	total_ordered integer,
    PRIMARY KEY ("SKU")
);


INSERT INTO PRODUCT_DETAILS 

("SKU", NAME, "orderedQuantity",	"stockLevel",	
"restockingLeadTime", "sentimentScore",	"sentimentMagnitude",
 TOTAL_ORDERED)
 
SELECT PRODUCT_DT.*,
	COALESCE(PRODUCT_SALES.TOTAL_ORDERED,
		0) AS TOTAL_ORDERED
FROM
		(SELECT *
		FROM PUBLIC.PRODUCTS
		UNION 
		 -- Since we don't have the details for the 2033 SKUs in 
		 -- sessions table we will be taking 0 as default value for
		 -- the same in the below query
		select "SKU",
		product_name_table.name,0 AS "orderedQuantity",
		0 AS "stockLevel",
		0 AS "restockingLeadTime",
		0.0 AS "sentimentScore",
		0.0 AS "sentimentMagnitude" from 
				(SELECT "productSKU" AS "SKU", "v2ProductName" as name,
				 row_number() over (partition by "productSKU" order by 
								   length("v2ProductName")) rnk
				 from PUBLIC.ALL_SESSIONS S) 
		 product_name_table
		 where rnk = 1
		 and "SKU" NOT IN
				(SELECT DISTINCT "SKU"
				FROM PUBLIC.PRODUCTS)) 
		 PRODUCT_DT
	
	LEFT JOIN 
	
	PUBLIC.SALES_BY_SKU PRODUCT_SALES 
	ON PRODUCT_DT."SKU" = PRODUCT_SALES."productSKU"
	
-- The primary key of the new products table will be the SKU
-- Now in the next steps we will add the ratio that was part of the 
-- sales_report table and clean the names of few of the columns. 

-- Below is the query to check for anomalies, duplicates, missing values


select * from product_details

-- Query to check for same name mapped to multiple SKUs
select name, count(*) 
from product_details 
group by name
order by 2 desc, length(name) desc

-- Check range of values in other columns

select min("orderedQuantity"), max("orderedQuantity"),
min("stockLevel"), max("stockLevel"),
min("restockingLeadTime"), max("restockingLeadTime"),
min("sentimentScore"), max("sentimentScore"),
min("sentimentMagnitude"), max("sentimentMagnitude"),
min(total_ordered), max(total_ordered)
from public.product_details

-- Some goods like Kick Ball which have sales order count of 3 and inventory 
-- of 723, have been ordered in large quantities ~15K 

-- Now we will rename the columns and create a final products table


create table cleaned_product_details as
select "SKU" as sku,
trim(both ' ' from name) as product_name,
"orderedQuantity" as supplier_ordered_quantity,
"stockLevel" as stock_level,
"restockingLeadTime" as restocking_lead_time,
"sentimentScore" as sentiment_score, 
"sentimentMagnitude" as sentiment_magnitude,
total_ordered as total_sales_orders 
from public.product_details;

ALTER TABLE cleaned_product_details ADD PRIMARY KEY 
(sku);

select * from public.cleaned_product_details

-- File -> Analytics

select * from public.analytics 

select count(*) from public.analytics  -- 4301122
-- We can see that there are lots of duplicates in the 
-- analytics table.
-- First step would be to make the table unique. 


create table public.non_duplicate_analytics_table as
select distinct * from public.analytics

select count(*) from public.non_duplicate_analytics_table
-- 1739308

select * from public.non_duplicate_analytics_table

-- Since there are lots of null values, we'll check the count of
-- missing values across columns and drop the ones with lots of 
-- missing values

select 
sum(case when "visitNumber" is null then 1 else 0 end) 
as visit_number_null,
sum(case when "visitId" is null then 1 else 0 end) 
as visit_id_null,
sum(case when "visitStartTime" is null then 1 else 0 end) 
as visitStartTime_null,
sum(case when date is null then 1 else 0 end) 
as date_null,
sum(case when "fullvisitorId" is null then 1 else 0 end) 
as fullvisitorId_null,
sum(case when userid is null then 1 else 0 end) 
as userid_null,
sum(case when "channelGrouping" is null then 1 else 0 end) 
as channelGrouping_null,
sum(case when "socialEngagementType" is null then 1 else 0 end) 
as socialEngagementType_null,
sum(case when units_sold is null then 1 else 0 end) 
as units_sold_null,
sum(case when pageviews is null then 1 else 0 end) 
as pageviews_null,
sum(case when timeonsite is null then 1 else 0 end) 
as timeonsite_null,
sum(case when bounces is null then 1 else 0 end) 
as bounces_null,
sum(case when revenue is null then 1 else 0 end) 
as revenue_null,
sum(case when unit_price is null then 1 else 0 end) 
as unit_price_null
from public.non_duplicate_analytics_table

-- There are lots of nulls in this table. Following are the steps
-- that we will take to clean the data. 
-- Since User ID is null for all records we can simply drop it. 

ALTER TABLE public.non_duplicate_analytics_table
DROP COLUMN userid;

select * from public.non_duplicate_analytics_table

-- To study the repetition of records for each user, we will 
-- add row_number to the table and check the reason for
-- repetition of records. Also, when we rank we will rank 
-- on revenue and the count of nulls in a row. Whichever row
-- has some value in revenue and low count of nulls will be given preference. 

DROP TABLE non_duplicate_analytics_table_with_row_number CASCADE;

create temp table non_duplicate_analytics_table_with_row_number
as 
select *, row_number() over (partition by 
					"fullvisitorId",date,
					"visitId",
					"visitStartTime",
					"visitNumber" order by
					revenue desc NULLS LAST, total_null_count) as row_num_nulls
from 							
		(select *, 
			(case when "visitNumber" is null then 1 else 0 end
			+
			case when "visitId" is null then 1 else 0 end
			+
			case when "visitStartTime" is null then 1 else 0 end
			+
			case when date is null then 1 else 0 end
			+
			case when "fullvisitorId" is null then 1 else 0 end
			+
			case when "channelGrouping" is null then 1 else 0 end
			+
			case when "socialEngagementType" is null then 1 else 0 end
			+
			case when units_sold is null then 1 else 0 end
			+
			case when pageviews is null then 1 else 0 end 
			+
			case when timeonsite is null then 1 else 0 end
			+
			case when bounces is null then 1 else 0 end 
			+
			case when revenue is null then 1 else 0 end
			+
			case when unit_price is null then 1 else 0 end )
			as total_null_count
			from public.non_duplicate_analytics_table) t1


select * from 
non_duplicate_analytics_table_with_row_number
where revenue is not null and "fullvisitorId" in 
(select distinct "fullVisitorId" from public.all_sessions)
order by "fullvisitorId", "visitId", date, 
"visitStartTime", "visitNumber", row_num_nulls

-- We can see that the total revenue in both the tables
-- is matching for few records. Example shared below - 

select * from public.all_sessions
where "fullVisitorId" = '0348842070964414318'
-- 71190000
select * from non_duplicate_analytics_table_with_row_number
where "fullvisitorId" = '0348842070964414318'
and revenue is not null
-- 71190000


-- Wherever we do not have the totalrevenue in all_sessions
-- we cannot validate the data with analytics table. But given that
-- total_transaction_revenue is matching with the analytics data
-- we can assume that wherever we have revenue and units sold
-- those records represents sales. The assumption is based on the fact
-- that sessions may not have the complete data. 

-- Since there are lots of repetitions in the analytics table
-- we will filter only those records wherever we have revenue and 
-- use that for further analysis. 

-- The below code includes cleaning code as well, in terms of
-- modifying the data types wherever required.

DROP TABLE public.revenue_analytics_table CASCADE;

create table public.revenue_analytics_table as
select "visitNumber" as visit_number, 
"visitId" as visit_id, 
TO_TIMESTAMP("visitId") as visit_date_time,
TO_DATE(date, 'YYYYMMDD') as visit_date,
"fullvisitorId" as full_visitor_id,
"channelGrouping" as sourcing_channel, 
"socialEngagementType" as engagement_type,
coalesce(units_sold, 0) as units_sold,
coalesce(pageviews, 0) as pageviews,
interval '1 second' * timeonsite as timeonsite,
coalesce(bounces, 0) as bounces,
revenue/1000000::real as revenue,
unit_price/1000000::real as unit_price
from public.analytics
where revenue is not null



-- File -> all_session_details


-- Check for anomalies, distributions, missing values etc.

select * from public.all_sessions
limit 5

-- Data type issues
-- Time and time on site has to be converted to timestamp from 
-- unix timestamp

select country, count(*)
from public.all_sessions
group by country
order by 2 desc -- 24 records have (not set) value

select city, count(*)
from public.all_sessions
group by city
order by 2 desc 
-- 8302 records have "not available in demo dataset"
-- and 354 have (not set) value

select count(*)
from public.all_sessions where time is null --0

select count(*)
from public.all_sessions where "timeOnSite" is null --3300

-- We can drop timeonsite since it's redundant and has null values


-- VisitID is in unix timestamp, it can be converted to date time

select 
"visitId" as visit_id, 
TO_TIMESTAMP("visitId") as visit_date_time
from public.all_sessions
limit 5

-- All the amount columns have to be divided by 1000000 (will
-- be incorporated later in the cleaning code)

select "productSKU", count(*) from
(select distinct "productSKU","v2ProductName"
from public.all_sessions) t1
group by "productSKU"
having count(*) > 1 
-- 74 SKUs with multiple names mapped

select "productSKU", count(*) from
(select distinct "productSKU","v2ProductCategory"
from public.all_sessions) t1
group by "productSKU"
having count(*) > 1 
-- 351 SKUs with multiple categories mapped

select "v2ProductCategory", count(*)
from public.all_sessions
group by "v2ProductCategory"
order by 2 desc 
-- (not set) - 757; 	
-- ${escCatTitle} - 19


select "productVariant", count(*)
from public.all_sessions
group by "productVariant"
order by 2 desc 
-- (not set) - 15094

select "transactionId", count(*)
from public.all_sessions
group by "transactionId"
-- 9 rows

select transactions, count(*)
from public.all_sessions
group by transactions
-- 81

-- transactionid and transactions column can be used to identify transactions

select "pagePathLevel1", count(*)
from public.all_sessions
group by "pagePathLevel1"
-- certain categories can be merged, merging logic will be covered in cleaning code

DROP TABLE public.cleaned_session_details CASCADE;

create table cleaned_session_details as
select "fullVisitorId" as full_visitor_id, 
"channelGrouping" as channel_grouping,
interval '1 millisecond' * time as time_spent_on_site, 
case 
when country = '(not set)' then 'Country Not Available' else 
country end
as country,
case 
when city in ('(not set)', 'not available in demo dataset') 
then 'City Not Available' else 
city end
as city,
"totalTransactionRevenue" as total_transaction_revenue,
transactions, 
--interval '1 second' * "timeOnSite" as time_on_site
-- TimeOnSite was dropped as it had 3300 Nulls plus it was 
-- redundant as we already have another time column without nulls
pageviews, "sessionQualityDim" as session_quality_dim,
--date, TO_DATE(date, 'YYYYMMDD') as date2,
-- date is dropped as we have visitId in unix timestamp format
-- from which we can derive the timestamp
"visitId" as visit_id, 
TO_TIMESTAMP("visitId") as visit_date_time,
type, "productRefundAmount" as product_refund_amount,
"productQuantity" as product_quantity,
"productPrice"/1000000::real as product_price,
"productRevenue"/1000000::real as product_revenue,
"productSKU" as sku,
trim(both ' ' from "v2ProductName") as product_name,
case 
when "v2ProductCategory" in ('(not set)', 
							 '${escCatTitle}') 
then 'Category Not Available' else 
trim(both ' ' from "v2ProductCategory") end as product_category,
case 
when "productVariant" in ('(not set)') 
then 'Variant Not Available' else 
trim(both ' ' from "productVariant") end as product_variant,
"currencyCode" as currency_code,
"itemQuantity" as item_quantity,
"itemRevenue"/1000000::real as item_revenue,
"transactionRevenue"/1000000::real as transaction_revenue,
"transactionId" as transaction_id,
"pageTitle" as page_title,
"searchKeyword" as search_keyword,
"pagePathLevel1" as page_path_level,
case 
when "pagePathLevel1" in ('/asearch.html', '/asearch.html/') then 'asearch.html'
when "pagePathLevel1" in ('/store.html', '/store.html/') then 'store.html'
when "pagePathLevel1" in ('/google+redesign/') then 'google+redesign'
when "pagePathLevel1" in ('/basket.html') then 'basket.html'
when "pagePathLevel1" in ('/ordercompleted.html') then 'ordercompleted.html'
when "pagePathLevel1" in ('/google+redesign/') then 'google+redesign'
when "pagePathLevel1" in ('/payment.html') then 'payment.html'
when "pagePathLevel1" in ('/revieworder.html') then 'revieworder.html'
when "pagePathLevel1" in ('/storeitem.html') then 'storeitem.html'
when "pagePathLevel1" in ('/yourinfo.html') then 'yourinfo.html'
end as page_path_level_revised,
"eCommerceAction_type" as ecommerce_action_type,
"eCommerceAction_step" as ecommerce_action_step,
"eCommerceAction_option" as ecommerce_action_option
from public.all_sessions

-- Add primary key and foreign key constraint

ALTER TABLE cleaned_session_details ADD PRIMARY KEY 
(full_visitor_id, time_spent_on_site, visit_id, sku);


ALTER TABLE cleaned_session_details
ADD CONSTRAINT fk_session_product
FOREIGN KEY (sku) REFERENCES cleaned_product_details(sku);

```
Finally we will be using 3 tables -- 
 1. cleaned_product_details
 2. cleaned_session_details
 3. revenue_analytics_table
