[1. Marketing Campaign Success [Advanced]](https://platform.stratascratch.com/coding/514-marketing-campaign-success-advanced?code_type=1)

You have the `marketing_campaign` table, which records in-app purchases by users. Users making their first in-app purchase enter a marketing campaign, where they see call-to-actions for more purchases. Find how many users made additional purchases due to the campaign's success.
The campaign starts one day after the first purchase. Users with only one or multiple purchases on the first day do not count, nor do users who later buy only the same products from their first day.
```sql
WITH users_filtered AS (
SELECT 
user_id
FROM (
SELECT
user_id, product_id, created_at,
first_value(created_at) OVER (PARTITION BY user_id ORDER BY created_at) as first_purchase
FROM marketing_campaign) sub
GROUP BY user_id
HAVING count(DISTINCT product_id) - count(DISTINCT product_id) FILTER (WHERE created_at =  first_purchase) > 0)

SELECT count(*) AS user_count FROM users_filtered
```
[2. Most Popular Client For Calls](https://platform.stratascratch.com/coding/2029-the-most-popular-client_id-among-users-using-video-and-voice-calls?code_type=1)

Select the most popular client_id based on a count of the number of users who have at least 50% of their events from the following list: 'video call received', 'video call sent', 'voice call received', 'voice call sent'.

```sql
WITH t as (
SELECT
user_id
FROM fact_events f
GROUP BY user_id
HAVING count(event_type) FILTER (WHERE event_type IN ('video call received',
                        'video call sent', 'voice call received', 'voice call sent')) * 1.0 / count(event_type) >= 0.5)
                        
SELECT
client_id
FROM t
INNER JOIN fact_events f USING(user_id)
GROUP BY client_id
ORDER BY count(*) DESC
LIMIT 1
```
[3. City With Most Amenities](https://platform.stratascratch.com/coding/9633-city-with-most-amenities?code_type=1)

You're given a dataset of searches for properties on Airbnb. For simplicity, let's say that each search result (i.e., each row) represents a unique host. Find the city with the most amenities across all their host's properties. Output the name of the city.

```sql
SELECT city
FROM airbnb_search_details
GROUP BY city
ORDER BY sum(array_length(STRING_TO_ARRAY(amenities, ','), 1)) DESC
LIMIT 1
```

[4. Top 5 States With 5 Star Businesses](https://platform.stratascratch.com/coding/10046-top-5-states-with-5-star-businesses?code_type=1)

Find the top 5 states with the most 5 star businesses. Output the state name along with the number of 5-star businesses and order records by the number of 5-star businesses in descending order. In case there are ties in the number of businesses, return all the unique states. If two states have the same result, sort them in alphabetical order.

```sql
SELECT state, n_businesses FROM (
SELECT 
state, 
count(stars) FILTER (WHERE stars = 5) AS n_businesses,
DENSE_RANK() OVER  (ORDER BY count(stars) FILTER (WHERE stars = 5) DESC) as rnk
FROM yelp_business
GROUP BY state
ORDER BY n_businesses DESC, state) sub
WHERE sub.rnk < 6
```
