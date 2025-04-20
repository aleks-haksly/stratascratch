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
[2. Number of Shipments Per Month](https://platform.stratascratch.com/coding/2056-number-of-shipments-per-month?code_type=1)

Write a query that will calculate the number of shipments per month. The unique key for one shipment is a combination of shipment_id and sub_id. Output the year_month in format YYYY-MM and the number of shipments in that month.

```sql
SELECT
  to_char(shipment_date, 'YYYY-MM') as year_month, 
  count(distinct shipment_id::char(3) || sub_id::char(1))  AS "count"
FROM amazon_shipment
GROUP BY to_char(shipment_date, 'YYYY-MM')
```


[3. Cookbook Recipes](https://platform.stratascratch.com/coding/2089-cookbook-recipes?code_type=1)

You are given a table containing recipe titles and their corresponding page numbers from a cookbook. Your task is to format the data to represent how recipes are distributed across double-page spreads in the book.


Each spread consists of two pages:


⦁   The left page (even-numbered) and its corresponding recipe title (if any).
⦁   The right page (odd-numbered) and its corresponding recipe title (if any).


The output table should contain the following three columns:


⦁   left_page_number – The even-numbered page that starts each double-page spread.
⦁   left_title – The title of the recipe on the left page (if available).
⦁   right_title – The title of the recipe on the right page (if available).


For the  k-th  row (starting from 0):


⦁   The  left_page_number  should be $2 * k$.
⦁   The  left_title  should be the title from page $2 * k$, or NULL if there is no recipe on that page.
⦁   The  right_title  should be the title from page $2 * k + 1$, or NULL if there is no recipe on that page.


Each page contains at most one recipe and  if a page does not contain a recipe, the corresponding title should be NULL. Page 0 (the inside cover) is always empty and included in the output. The table should ensure that all pages up to the maximum recorded page number are included, even if they contain

```sql
WITH series AS
  (SELECT generate_series AS page_number
   FROM generate_series(0,   (SELECT max(page_number) FROM cookbook_titles))),
   
pages as (SELECT s.page_number,
c.title
FROM series s
LEFT JOIN cookbook_titles c ON s.page_number = c.page_number)

 
SELECT (row_number() over(ORDER BY page_number/2)-1)*2 AS left_page_number,
string_agg(case when page_number % 2 = 1 THEN title END, ',') AS right_title,
string_agg(case when page_number % 2 = 0 THEN title END, ',') AS left_title

FROM pages
GROUP BY (page_number / 2)
```
