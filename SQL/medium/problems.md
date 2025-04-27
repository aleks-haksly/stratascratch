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
[4. Premium Accounts](https://platform.stratascratch.com/coding/2097-premium-acounts?code_type=1)

You have a dataset that records daily active users for each premium account. A premium account appears in the data every day as long as it remains premium. However, some premium accounts may be temporarily discounted, meaning they are not actively paying—this is indicated by a final_price of 0.


For each of the first 7 available dates, count the number of premium accounts that were actively paying on that day. Then, track how many of those same accounts remain premium and are still paying exactly 7 days later (regardless of activity in between).


Output three columns:
•   The date of initial calculation.
•   The number of premium accounts that were actively paying on that day.
•   The number of those accounts that remain premium and are still paying after 7 days.

```sql
WITH t as (select 
account_id, entry_date,
entry_date + INTERVAL  '7 days' as day7_after
FROM premium_accounts_by_day
WHERE final_price > 0),

t2 as (
SELECT 
t.entry_date, 
COUNT(t.account_id) as premium_paid_accounts_after_7d
FROM premium_accounts_by_day p
INNER JOIN t ON p.account_id = t.account_id AND p.entry_date = t.day7_after
WHERE p.final_price > 0
GROUP By t.entry_date)

SELECT 
entry_date, 
COUNT(account_id) as premium_paid_accounts, 
MAX(t2.premium_paid_accounts_after_7d) as premium_paid_accounts_after_7d
FROM premium_accounts_by_day
INNER JOIN t2 USING(entry_date)
WHERE final_price > 0
GROUP BY entry_date
ORDER BY 1
```
[5. Election Results](https://platform.stratascratch.com/coding/2099-election-results?code_type=1)

The election is conducted in a city and everyone can vote for one or more candidates, or choose not to vote at all. Each person has 1 vote so if they vote for multiple candidates, their vote gets equally split across these candidates. For example, if a person votes for 2 candidates, these candidates receive an equivalent of 0.5 vote each. Some voters have chosen not to vote, which explains the blank entries in the dataset.


Find out who got the most votes and won the election. Output the name of the candidate or multiple names in case of a tie.
To avoid issues with a floating-point error you can round the number of votes received by a candidate to 3 decimal places.

```sql
WITH vp as (
SELECT 
voter,
round(1.0 / NULLIF(count(candidate), 0), 3) as vote_power
FROM voting_results
GROUP BY voter),

ranked as (
SELECT
candidate, 
dense_rank() OVER (ORDER BY sum(vote_power) DESC) as rnk
FROM voting_results vr
INNER JOIN vp USING(voter)
WHERE vp.vote_power IS NOT NULL AND vr.candidate IS NOT NULL
GROUP BY candidate)

SELECT candidate FROM ranked WHERE rnk = 1
```

[6. Flags per Video](https://platform.stratascratch.com/coding/2102-flags-per-video?code_type=1)

For each video, find how many unique users flagged it. A unique user can be identified using the combination of their first name and last name. Do not consider rows in which there is no flag ID.

```sql
SELECT
video_id, count(DISTINCT COALESCE(user_firstname, '') || COALESCE(user_lastname, '')) AS num_unique_users
from user_flags
WHERE flag_id IS NOT NULL
GROUP BY video_id
```
[User with Most Approved Flags](https://platform.stratascratch.com/coding/2104-user-with-most-approved-flags?code_type=1)

Which user flagged the most distinct videos that ended up approved by YouTube? Output, in one column, their full name or names in case of a tie. In the user's full name, include a space between the first and the last name.

```sql
WITH approved_flags AS (
SELECT * FROM user_flags uf
INNER JOIN flag_review fr USING(flag_id)
WHERE fr.reviewed_outcome = 'APPROVED'),

rnaked AS (
SELECT 
user_firstname || ' ' || user_lastname AS username,
dense_rank() OVER (ORDER BY count(DISTINCT video_id) DESC) as rnk
FROM approved_flags
GROUP BY user_firstname || ' ' || user_lastname)

SELECT username FROM rnaked WHERE rnk = 1
```
[Most Lucrative Products (easy)](https://platform.stratascratch.com/coding/2119-most-lucrative-products?code_type=1)
You have been asked to find the 5 most lucrative products in terms of total revenue for the first half of 2022 (from January to June inclusive).
Output their IDs and the total revenue.

```sql
WITH ranked_products as (
SELECT
product_id,
SUM(cost_in_dollars * units_sold) as revenue,
dense_rank() OVER (ORDER BY SUM(cost_in_dollars * units_sold) DESC) as rnk
FROM online_orders
WHERE date_trunc('month', date_sold) BETWEEN '2022-01-01' AND '2022-07-01'::DATE - INTERVAL '1 day'
GROUP BY product_id)

SELECT 
product_id,
revenue
FROM ranked_products
WHERE rnk < 6
```

[7. Find Students At Median Writing](https://platform.stratascratch.com/coding/9610-find-students-with-a-median-writing-score?code_type=1)

Identify the IDs of students who scored exactly at the median for the SAT writing section.

```sql
WITH median as (
SELECT
percentile_cont(0.5) WITHIN GROUP (ORDER BY sat_writing ASC) as med
FROM sat_scores)

SELECT 
student_id
FROM sat_scores
WHERE sat_writing = (SELECT med FROM median)
```
[8. Host Popularity Rental Prices](https://platform.stratascratch.com/coding/9632-host-popularity-rental-prices?code_type=1)

You are given a table named airbnb_host_searches that contains data for rental property searches made by users. Your task is to determine the minimum, average, and maximum rental prices for each host’s popularity rating.


The host’s popularity rating is defined as below:
•   0 reviews: "New"
•   1 to 5 reviews: "Rising"
•   6 to 15 reviews: "Trending Up"
•   16 to 40 reviews: "Popular"
•   More than 40 reviews: "Hot"


Tip: The id column in the table refers to the search ID.


Output host popularity rating and their minimum, average and maximum rental prices. Order the solution by min_price.

```sql
WITH pop as (
select
id, 
CASE WHEN number_of_reviews = 0 THEN 'New'
     WHEN number_of_reviews <= 5 THEN 'Rising'
     WHEN number_of_reviews <= 15 THEN 'Trending Up'
     WHEN number_of_reviews <= 40 THEN 'Popular'
     ELSE 'Hot' END AS host_popularity 
from airbnb_host_searches)

SELECT 
pop.host_popularity,
min(price) as min_price,
avg(price) as avg_price,
max(price) as max_price
FROM airbnb_host_searches a
INNER JOIN pop USING(id)
GROUP BY pop.host_popularity
ORDER BY 2
```
