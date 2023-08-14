## Task 1

### Query
```sql
WITH events(id, occur_time, close_time) AS (
    VALUES
    (1, '09:00:00', '10:00:00'),
    (1, '09:00:00', '09:20:00'),
    (1, '09:10:00', '09:50:00'),
    (1, '08:55:00', '09:50:00'),
    (1, '08:00:00', '08:51:00'),
    (1, '08:10:00', '08:50:00'),
    (1, '09:10:00', '10:50:00'),
    (2, '08:00:00', '08:55:00'),
    (2, '08:10:00', '08:50:00')
),
tmp AS (
    SELECT id, occur_time, close_time, 
    ROW_NUMBER() OVER (PARTITION BY id ORDER BY occur_time) AS event_order,
    LAG(close_time,1) OVER (PARTITION BY id ORDER BY occur_time) last_close_time
    FROM events
    ),
tmp2 AS (
    SELECT * , 
    last_close_time < occur_time,
    ROW_NUMBER() OVER (PARTITION BY id ,last_close_time < occur_time) AS rank_
    FROM tmp
    ORDER BY 1,2
    ),
tmp3 AS (
    SELECT *, 
    CASE WHEN last_close_time IS NULL THEN 1 ELSE event_order - rank_ END AS task_id
    FROM tmp2
)
SELECT id, task_id, MIN(occur_time) AS occur_time, MAX(close_time) as close_time
FROM tmp3
GROUP BY 1,2
ORDER BY 1,2
;
```
### Output
![image](https://github.com/JoyceZhou96/ELITE-Data-Analyst-Assignment/assets/51615886/005c403d-d85f-44a0-8dd7-f9eebf47bb61)

## Task 2
Combine 2 sub tasks into one [query](https://console.cloud.google.com/bigquery?sq=208774998227:f659176e31cb421b8ff87773c0fe1ea0)
### Query
```sql
with login_base as (
  select user_pseudo_id as user_id,
  event_date as login_date
  from `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
),
tmp as (
  select a.login_date as initial_visit_date,
  a.user_id as user_id,
  date_diff(PARSE_DATE('%Y%m%d', b.login_date), PARSE_DATE('%Y%m%d', a.login_date), DAY) as days,
  b.user_id as retain_user
  from login_base a 
  left join login_base b 
  on a.user_id = b.user_id and a.login_date < b.login_date
  group by 1,2,3,4
),
tmp2 as (
  select initial_visit_date, 
    count(distinct user_id) as users
  from tmp
  group by 1
),
tmp3 as (
  select initial_visit_date, 
    days,
  count(distinct retain_user) as retained
from tmp
group by 1,2)
select a.initial_visit_date, a.users, b.days, b.retained, b.retained*1.0000/a.users as retention_rate
from tmp2 a 
left join tmp3 b 
on a.initial_visit_date = b.initial_visit_date
```

## Task 3
<img width="502" alt="image" src="https://github.com/JoyceZhou96/ELITE-Data-Analyst-Assignment/assets/51615886/47b454b6-6045-4e4b-bd47-70136e6c7db5">

Explanation: Since the dashboard cannot be saved to obtain a link, please take a screenshot of the dashboard for reference.

### Chart 1: Daily Login Users
This chart illustrates the overall number of active users on a daily basis, and it reveals a downward trend.

### Chart 2: Daily N+1 Retention Rate
This chart represents the retention rate of users from their initial visit to the subsequent day. It is apparent that the retention rate is also declining.

### Chart 3: Daily Retention Rate on Day X
This chart depicts the retention rate of users who return on a specific day. As time progresses, it becomes less probable for them to come back.
