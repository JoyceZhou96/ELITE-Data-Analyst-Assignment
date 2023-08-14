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




