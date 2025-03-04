# SQL_netflix_dataset_analysis

## Netflix Data Analysis Project Report

## Project Overview

This project aims to analyze Netflix's viewing data using SQL queries on PostgreSQL 17. The dataset includes movie information, user ratings, and watch history. The analysis explores various aspects of user behavior, movie performance, and engagement patterns using advanced SQL queries, including window functions.
- **Difficulty:** Advanced
- **Database:**'netflix_db' 

## Objective
1. The primary objectives of this project are:
2. Identify popular and least-watched movies.
3. Analyze user engagement based on watch time and ratings.
4. Examine subscription-based viewing patterns.
5. Investigate regional and demographic trends in content consumption.
6. Utilize window functions to rank and categorize data effectively.

## Project Structure
**-- Creating table for movies details**
```sql
DROP TABLE IF EXISTS movies;
CREATE TABLE movies (
	movie_id INT PRIMARY KEY,
	title VARCHAR,
	type VARCHAR,
	release_year VARCHAR,
	genre VARCHAR,	
	duration VARCHAR,
	language VARCHAR,
	country VARCHAR,
	imdb_rating VARCHAR
)
```
**-- Checking if data has been imported correctly or not**
```sql
SELECT * FROM movies
LIMIT 10;
```
**-- Checking count of all imported queries**
```sql
SELECT COUNT(*) FROM movies
```
**-- Creating table for user details**
```sql
DROP TABLE IF EXISTS user_details;
CREATE TABLE user_details (
	user_id INT,
	name VARCHAR,
	age INT,
	subscription_plan VARCHAR,
	join_date DATE,
	country VARCHAR
)
```
**-- Checking if data has been imported correctly or not**
```sql
SELECT * FROM user_details
LIMIT 10;
```
**-- Checking count of all imported queries**
```sql
SELECT COUNT(*) FROM user_details
```
**-- Creating table for ratings**
```sql
DROP TABLE IF EXISTS ratings;
CREATE TABLE ratings (
	rating_id INT,	
	user_id INT,	
	movie_id INT,
	rating INT,	
	rating_date DATE
)
```
**-- Checking if data has been imported correctly or not**
```sql
SELECT * FROM ratings
LIMIT 10;
```
**-- Checking count of all imported queries**
```sql
SELECT COUNT(*) FROM ratings
```
**-- Creating table for watch history**
```sql
DROP TABLE IF EXISTS watch_history;
CREATE TABLE watch_history (
	watch_id INT,
	user_id INT,
	movie_id INT,
	watch_date DATE,
	watch_time_minutes INT
)
```
**-- Checking if data has been imported correctly or not**
```sql
SELECT * FROM watch_history
LIMIT 10;
```
**-- Checking count of all imported queries**
```sql
SELECT COUNT(*) FROM watch_history
```
## DATA ANALYSIS
## **Advanced Level questions**

**1. Find the top 5 most-watched movies of all time.**
```sql
SELECT 
	m.title,
	COUNT(wh.watch_id) AS total_watches
FROM watch_history wh
LEFT JOIN movies m USING(movie_id)
GROUP BY m.title
ORDER BY total_watches DESC
LIMIT(5);
```
**2. Get the average IMDb rating of movies and TV shows separately.**
```sql
SELECT 
	type AS Category,
	ROUND(AVG(imdb_rating::NUMERIC), 1) AS imdb_rating
FROM movies
GROUP BY type
```
**3. Identify the top 3 highest-rated movies by Premium subscribers.**
```sql
WITH premium_rating AS 
(
	SELECT 
		m.title,
		ROUND(AVG(r.rating), 1) AS avg_rating
	FROM user_details ud
	INNER JOIN ratings r USING(user_id)
	INNER JOIN movies m USING(movie_id)
	WHERE ud.subscription_plan = 'Premium'
	GROUP BY m.title
)

SELECT * FROM (
	SELECT 
		title, 
		avg_rating,
		RANK() OVER (ORDER BY avg_rating DESC) AS ranking
	FROM premium_rating
) ranked_movies
WHERE ranking <= 3;
```
**4. List the most-watched genre by country.**
```sql
SELECT 
	m.genre,
	ud.country
FROM movies m 
INNER JOIN user_details ud USING(country)
INNER JOIN watch_history wh USING(user_id)
GROUP BY m.genre, ud.country
```
**5. Find the movie with the longest watch time in a single session.**
```sql
SELECT 
    m.title, 
    wh.user_id, 
    wh.watch_time_minutes
FROM watch_history wh
INNER JOIN movies m USING(movie_id)
ORDER BY wh.watch_time_minutes DESC
LIMIT 1;
```
**6. Determine the 10 most active users (based on total watch time).**
```sql
SELECT 
	ud.user_id,
	ud.name,
	SUM(wh.watch_time_minutes) AS total_watch_time
FROM watch_history wh
INNER JOIN user_details ud USING(user_id)
GROUP BY ud.user_id, ud.name
ORDER BY total_watch_time DESC
LIMIT 10;
```
**7. Calculate the average watch time per user by subscription plan.**
```sql
SELECT
	ud.user_id,
	ud.subscription_plan,
	ROUND(AVG(wh.watch_time_minutes), 2) AS avg_watch_time
FROM user_details ud
INNER JOIN watch_history wh USING(user_id)
GROUP BY ud.subscription_plan, ud.user_id
ORDER BY avg_watch_time DESC
```
**8. Get the percentage of users who have watched at least one TV show.**
```sql
WITH users_who_watched_tv AS (
	SELECT DISTINCT wh.user_id
	FROM watch_history wh
	INNER JOIN movies m USING(movie_id)
	WHERE m.type = 'TV Show'
	),
total_users AS (
	SELECT 
		COUNT(*) AS total
	FROM user_details)

SELECT 
	ROUND(((COUNT(uwt.user_id) * 100) / (SELECT total FROM total_users)), 2) AS percentage_user_who_watched_tv_shows
FROM users_who_watched_tv uwt
CROSS JOIN total_users tu
```
**9. Find the year with the highest number of movies released.**
```sql
SELECT 
	release_year,
	(SELECT
		COUNT(movie_id)
	FROM movies) AS no_of_movies,
	RANK() OVER(ORDER BY release_year) AS ranking
FROM movies
GROUP BY release_year, no_of_movies
ORDER BY no_of_movies DESC
```
**10. Identify movies that have an average user rating higher than their IMDb rating.**
```sql
WITH user_rated_movies AS (
		SELECT 
			m.title AS title,
			ROUND(AVG(r.rating), 1) AS avg_ratings
		FROM ratings r
		INNER JOIN movies m USING(movie_id)
		GROUP BY m.title
)

SELECT 
	m.title,
	urm.avg_ratings,
	m.imdb_rating
FROM user_rated_movies urm 
INNER JOIN movies m USING(title)
WHERE ((urm.avg_ratings::NUMERIC) > (m.imdb_rating::NUMERIC))
```
**11. Get the top 3 countries with the highest Netflix engagement (total watch time).**
```sql
WITH total_watch_time AS (
	SELECT
		ud.country,
		SUM(wh.watch_time_minutes) AS total_watch_time
	FROM watch_history wh
	INNER JOIN user_details ud USING(user_id)
	GROUP BY ud.country
	)

SELECT * FROM (
	SELECT 
		country,
		twt.total_watch_time,
		RANK() OVER (ORDER BY twt.total_watch_time) AS ranking
	FROM total_watch_time twt
	) ranked_countries
WHERE ranking <= 3;
```
**12. Find the number of unique users who watched each movie.**
```sql
SELECT
	movie_id,
	COUNT(DISTINCT user_id) AS watched_users
FROM watch_history 
GROUP BY movie_id
```
**13. Identify the top 5 least-watched movies.**
```sql
WITH watch_time AS (
	SELECT 
		m.title,
		SUM(wh.watch_time_minutes) AS total_watchtime
	FROM watch_history wh
	INNER JOIN movies m USING(movie_id)
	GROUP BY m.title
),
least_watched AS (
	SELECT 
		title,
		total_watchtime,
		RANK() OVER (ORDER BY total_watchtime ASC) AS ranking
	FROM watch_time
) 
SELECT 
	title,
	total_watchtime,
	ranking
FROM least_watched
WHERE ranking <= 5;
```
**14. Determine the user who has rated the most content.**
```sql
SELECT 
	user_id,
	COUNT(rating) AS total_ratings
FROM ratings
GROUP BY user_id
ORDER BY total_ratings DESC
LIMIT 1;
```
**15. Find three most common movie duration watched by users.**
```sql
WITH watched_duration AS (
	SELECT 
		m.duration AS movie_duration,
		COUNT(watch_time_minutes) AS count_occurance
	FROM watch_history wh
	INNER JOIN movies m USING(movie_id)
	WHERE m.type = 'Movie'
	GROUP BY m.duration
	),
ranked_durations AS (
	SELECT 
		movie_duration,
		count_occurance,
		RANK() OVER(ORDER BY count_occurance DESC) AS ranking
	FROM watched_duration
	)
SELECT 
	movie_duration,
	count_occurance,
	ranking
FROM ranked_durations
WHERE ranking <= 3;
```
**16. Get a list of users who have never rated any content.**
```sql
SELECT
	ud.name,
	r.rating
FROM ratings r
LEFT JOIN user_details ud USING(user_id)
WHERE rating IS NULL						-- there is not a single user who never rated any contents in the data
```
**17. Calculate the average rating given by users aged 18-25.**
```sql
SELECT 
	ROUND(AVG(r.rating), 1) AS avg_rating_18_25_age
FROM ratings r
INNER JOIN user_details ud USING(user_id)
WHERE ud.age BETWEEN 18 AND 25
```
**18. Identify users who watched a movie of a different country than their registered country.**
```sql
SELECT
	ud.name,
	ud.country AS user_country,
	m.title,
	m.country AS movie_country
FROM user_details ud
LEFT JOIN movies m USING(country)
LEFT JOIN watch_history wh USING(movie_id)
WHERE ud.country <> m.country    -- there is not a single user who watched movie from different country than it's registered country
```
**19. Find the movie with the highest average rating given by Standard plan users.**
```sql
SELECT
	m.title,
	ROUND(AVG(r.rating), 1) AS avg_rating,
	ud.subscription_plan
FROM movies m
INNER JOIN ratings r USING(movie_id)
INNER JOIN user_details ud USING(user_id)
WHERE ud.subscription_plan = 'Standard'
GROUP BY m.title, ud.subscription_plan
ORDER BY avg_rating DESC
LIMIT 1;	
```
**20. Determine the percentage of movies watched that were released in the last 5 years.**
```sql
WITH movies_watch_5_years AS (
	SELECT 
		COUNT(*) AS number_of_movies
	FROM movies
	INNER JOIN watch_history wh USING(movie_id)
	WHERE release_year BETWEEN '2020' AND '2025' 
),
total_movies_watched AS (
	SELECT 
		COUNT(*) AS total_watch
	FROM watch_history
)

SELECT 
	mw.number_of_movies,
	tmw.total_watch,
	ROUND((mw.number_of_movies * 100)/ NULLIF(tmw.total_watch, 0), 2) AS percent_watch_5_years
FROM movies_watch_5_years mw
CROSS JOIN total_movies_watched tmw;
```
## Findings

**1. Most-Watched and Least-Watched Movies**
The top 5 most-watched movies were identified using total watch count.
The least-watched movies were ranked based on total watch time.
**2. User Engagement and Ratings**
The most active users were determined based on total watch time.
Premium subscribers tend to give higher ratings compared to other subscription plans.
The movie with the highest average rating from Standard plan users was identified.
**3. Subscription-Based Viewing Patterns**
The average watch time per user was analyzed across different subscription plans.
A percentage of users who watched at least one TV show was calculated.
**4. Content Trends by Year and Genre**
The most common movie duration watched by users was identified.
The year with the highest number of movie releases was determined.
The most-watched genre was analyzed by country.
**5. Regional and Demographic Insights**
Users who watched content from a different country than their registered location were identified.
The percentage of movies watched that were released in the last five years was calculated.
Countries with the highest Netflix engagement were ranked based on total watch time.

## Report Highlights

- Top 3 highest-rated movies by Premium subscribers were determined using window functions.
- The most common movie durations were analyzed to understand user preferences.
- User activity rankings helped identify the most engaged users.
- Ratings vs IMDb scores provided insights into user perception of content quality.
- Demographic-based viewing patterns revealed trends among different age groups and subscription plans.

## Conclusion

This analysis provides valuable insights into Netflix's user behavior and content performance. The findings highlight key trends in content popularity, user engagement, and subscription-based viewing patterns. By leveraging SQL queries and window functions, we effectively ranked, filtered, and analyzed large datasets to uncover actionable insights. These results can help Netflix optimize its content strategy, enhance user experience, and improve recommendations based on viewing patterns.

## How to Use This Project on GitHub

- Clone the repository:
git clone https://github.com/NikhilD-Engineer/SQL_netflix_dataset_analysis.git

- Import the dataset into PostgreSQL 17.

- Run the SQL queries provided in the repository.

- Modify or expand queries to explore additional insights.

For further improvements, consider integrating this analysis with visualization tools like Tableau or Power BI!

## Author - Nikhil Dagale

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, or feedback, or would like to collaborate, feel free to get in touch!
The datasets used in this project are synthetic datasets generated using ChatGPT.
