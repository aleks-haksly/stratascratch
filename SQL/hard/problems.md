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
