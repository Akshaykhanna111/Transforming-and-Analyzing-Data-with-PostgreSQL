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

Question 2: 

SQL Queries:

Answer:



Question 3: 

SQL Queries:

Answer:



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
