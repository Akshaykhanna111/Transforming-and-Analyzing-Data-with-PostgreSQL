Question 1: Analyze the web traffic by country and channel grouping

SQL Queries:

```sql
select channel_grouping, count(*)
from
(select distinct channel_grouping, full_visitor_id
from public.cleaned_session_details
where ) t1
group by channel_grouping


select country, count(*)
from
(select distinct country, full_visitor_id
from public.cleaned_session_details) t1
group by country
order by 2 desc
limit 6

```

Answer: 
US has the largest number of visitors on the platform in terms of overall count followed by India and UK. 
Organic traffic contributes to about 59% of the overall volume, followed by Direct and Referral visitors. 

Question 2: Conversion rate analysis by Channel grouping and country ?

SQL Queries:

```sql
select channel_grouping, (sum(Conversion_flag)::real/count(*)::real) * 100
from
(select distinct channel_grouping, full_visitor_id,
 visit_id,
 case when TRANSACTION_ID like '%ORD%'
OR TRANSACTIONS = '1' then 1 else 0 end as Conversion_flag
from public.cleaned_session_details) t1
group by channel_grouping
order by 2 desc
limit 6

-- Conversion Rate by Country

select country, (sum(Conversion_flag)::real/count(*)::real) * 100
from
(select distinct country, full_visitor_id,
  visit_id,
 case when TRANSACTION_ID like '%ORD%'
OR TRANSACTIONS = '1' then 1 else 0 end as Conversion_flag
from public.cleaned_session_details) t1
group by country
order by 2 desc
limit 6

```
Answer:
Top countries from conversion stand point are Israel, Switzerland and US. Top channels in terms of conversion rates are Referral, Direct and Paid Search. 



Question 3: Analyze web traffic data by day, month, year and highlight if any insights found.

SQL Queries:
```sql

-- Year Month wise trends

select concat(to_char(visit_date_time, 'YYYY'), '-',
to_char(visit_date_time, 'Mon')) as Year-Month
count(*) as total_count_visits
from public.cleaned_session_details
group by concat(to_char(visit_date_time, 'YYYY'), '-',
to_char(visit_date_time, 'Mon') as Month)
order by 1

-- Day wise trends

select 
to_char(visit_date_time, 'Day') as Day,
count(*) as total_count_visits
from public.cleaned_session_details
group by to_char(visit_date_time, 'Day')

-- Hour wise trends

select 
to_char(visit_date_time, 'HH24') as Hour,
count(*) as total_count_visits
from public.cleaned_session_details
group by to_char(visit_date_time, 'HH24')
order by 1

```

Answer: Traffic volume is standard during weekdays around 15-16% whereas on weekends the volume dips to 10%. Probably more campaigns shall be executed during weekdays compared to weekends. Over a period of time the web traffic has been coming down as seen in the month-year trend analysis. The web traffic peaks during the day time between 2:00 pm to 7:00 pm. 

Question 4:List the top 10 and bottom 10 products as per the product sentiment

SQL Queries:

```sql
-- Top 5 products

select product_name, sentiment_score
from public.cleaned_product_details 
where sentiment_score is not null
order by 2 desc limit 10

-- Bottom 5 products

select product_name, sentiment_score
from public.cleaned_product_details 
where sentiment_score is not null
order by 2 limit 10


```

Answer: Products in the bottom 10 list shall be kept only during their demand season and should be dropped at the rest of times as it's leading to unnecessary operational costs and certainly bad sentiment amongst the customers who bought the same. 

Question 5: List down top 3 categories in each of the quarter to study the seasonality of categories throughout the year. 

SQL Queries:

```sql
select * from 
(select *, 
row_number() over (partition by Quarter
				  order by total_count_visits desc Nulls last) as ranks
from 
(select product_category, 
to_char(visit_date_time, 'Q') as Quarter,
count(*) as total_count_visits
from public.cleaned_session_details
 where product_category <> 'Category Not Available'
group by product_category, 
to_char(visit_date_time, 'Q')) t) t2
where 
ranks <= 3

```

Answer:Apparel is the most popular category across quarters, followed by Electronics and office goods. Knowing customer preferences for the season will help in reducing inventory management and supply chain costs by accordingly spending resources on the required goods. 
