## 550. Game Play Analysis IV

<pre>
Table: Activity

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
(player_id, event_date) is the primary key of this table.
This table shows the activity of players of some game.
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on some day using some device.
 

Write an SQL query that reports the fraction of players that logged in again on the day after the day they first logged in, rounded to 2 decimal places. 
In other words, you need to count the number of players that logged in for at least two consecutive days starting from their first login date, 
then divide that number by the total number of players.

The query result format is in the following example:

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
Only the player with id 1 logged back in after the first day he had logged in so the answer is 1/3 = 0.33

</pre>

------------------------------------------------------------------------------
## Solution

### Solution 2
------------------------------------------------------------------------------
```sql
# Write your MySQL query statement below
/*
The idea is to use left join
*/

SELECT ROUND((COUNT(t2.player_id))/(COUNT(t1.player_id)),2) AS 'fraction'
FROM (
    SELECT a.player_id, MIN(a.event_date) AS 'first_time'
    FROM Activity a
    GROUP BY a.player_id
     ) AS t1
LEFT JOIN (
    SELECT a.player_id, a.event_date
    FROM Activity a
    ) AS t2
ON t1.player_id = t2.player_id
   AND DATEDIFF(t1.first_time, t2.event_date) = -1

```
------------------------------------------------------------------------------
### Solution 1
```sql

# Write your MySQL query statement below

WITH t1 AS (
    SELECT COUNT(DISTINCT a0.player_id) AS Num
                FROM Activity a0
                WHERE (a0.event_date - (SELECT MIN(a1.event_date)
                                            FROM Activity a1
                                            WHERE a0.player_id = a1.player_id
                                          )) = 1
                                           
                                           
)
,
t2 AS(
                SELECT COUNT(DISTINCT a0.player_id) AS Num
                FROM Activity a0   
)

SELECT (Cast((t1.Num/t2.Num) AS DECIMAL(12,2))) AS 'fraction'
FROM t1, t2


```
