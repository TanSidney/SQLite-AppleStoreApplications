CREATE Table appleStore_description_combined AS

SELECT * FROM appleStore_description1

UNION ALL

SELECT * FROM appleStore_description2

UNION ALL

SELECT * FROM appleStore_description3

UNION ALL

SELECT * FROM appleStore_description4


SELECT * FROM AppleStore



**EXPLORATORY DATA ANALYSIS**

 --CHECK NUMBER OF UNIQUE APPS IN BOTH TABLES-- 
 
SELECT COUNT(DISTINCT id) AS UniqueAppIDs
FROM AppleStore
 
SELECT COUNT(DISTINCT id) AS UniqueAppIDs
FROM appleStore_description_combined

 --CHECK FOR ANY MISISING VALUES IN KEY FIELDS--
 
SELECT COUNT(*) AS MissingValues
FROM AppleStore
WHERE track_name is NULL OR user_rating IS NULL OR prime_genre is NULL

SELECT COUNT(*) AS MissingValues
FROM appleStore_description_combined
WHERE app_desc is NULL

 --FIND OUT THE NUMBER OF APPS PER GENRE--AppleStore
 
 SELECT prime_genre, COUNT(*) AS NumApps
 FROM AppleStore
 GROUP BY prime_genre
 ORDER BY NumApps DESC
 
 --GET AN OVERVIEIW OF THE APPS' RATINGS--
 
SELECT 
min(user_rating) AS MinRating, 
max(user_rating) AS MaxRating, 
avg(user_rating) AS AvgRating
FROM AppleStore

--GET THE DISTRIBUTION OF APP PRICES--

SELECT
	(price / 2) *2 AS PriceBinStart,
    ((price / 2) *2) +2 As PriceBinEnd,
    COUNT(*) AS NumApps
FROM AppleStore
group by PriceBinStart
Order by PriceBinStart

**DATA ANALYSIS**

--Determine whether paid apps have higher ratings than free appsAppleStore

SELECT CASE
			when price > 0 THEN 'Paid'
            ELSE 'Free'
		end as App_Type,
        avg(user_rating) as Avg_Rating
FROM AppleStore
GROUP BY App_Type


--CHECK IF APPS WITH MOORE SUPPORTED LANGUAGES HAVE HIGHER RATINGSAppleStore

SELECT CASE
			WHEN lang_num < 10 THEN '<10 languges'
            WHEN lang_num BETWEEN 10 and 30 then '10-30 languages'
            else '>30 languages'
		end as language_bucket,
        avg(user_rating) as Avg_Rating
FROM AppleStore
GROUP BY language_bucket
ORDER by Avg_Rating DESC

--CHECK GENRES WITH LOW RATINGS--

SELECT prime_genre,
	   avg(user_rating) as Avg_Rating
FROM AppleStore
group by prime_genre
order by Avg_Rating asc
LIMIT 10

--CHECK IF THERE IS CORRELATION BETWEEN THE LENGHT OF THE APP DESCRIPTION AND THE USER RATING--

SELECT CASE
			WHEN length(b.app_desc) <500 Then 'Short'
			WHEN length(b.app_desc) BETWEEN 500 AND 1000 THEN 'Medium'
            ELSE 'Long'
		end as description_lenght_bucket,
        avg(a.user_rating) AS average_rating
FROM
	AppleStore AS a
JOIN
	appleStore_description_combined AS b
ON 
	a.id = b.id

GROUP BY description_lenght_bucket
ORDER BY average_Rating DESC

--CHECK THE TOP-RATED APPS FOR EACH GENRE--AppleStore

SELECT
	prime_genre,
    track_name,
    user_rating
from ( 
		SELECT
  		prime_genre,
 		track_name,
  		user_rating,
  		RANK() OVER(PARTITION BY prime_genre ORDER BY user_rating DESC, rating_count_tot DESC) AS rank
  		FROM 
  		AppleStore
  	  ) AS a 
 WHERE
 a.rank = 1
