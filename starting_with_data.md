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

Answer:



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
