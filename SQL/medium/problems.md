[1. Users By Average Session Time](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=1)

Calculate each user's average session time, where a session is defined as the time difference between a page_load and a page_exit. Assume each user has only one session per day. If there are multiple page_load or page_exit events on the same day, use only the latest page_load and the earliest page_exit, ensuring the page_load occurs before the page_exit. Output the user_id and their average session time.

```sql
WITH last_loads as (
select DISTINCT 
user_id,
date_trunc('day', timestamp) as date,
max(timestamp) as last_load
from facebook_web_log
WHERE action = 'page_load'
GROUP BY user_id, date_trunc('day', timestamp)),

users_deltas as (
SELECT 
fl.user_id, ll.date,
min(fl.timestamp) - max(last_load) as delta
FROM facebook_web_log fl
INNER JOIN last_loads ll ON fl.user_id = ll.user_id AND  date_trunc('day', fl.timestamp) = ll.date AND fl.timestamp > last_load AND fl.action = 'page_exit'
GROUP BY fl.user_id, ll.date)


SELECT
user_id,
AVG(delta) as avg_session_duration
FROM users_deltas
GROUP BY user_id;
```
