# NETFLIX MOVIES AND TV SHOWS DATA ANALYSIS USING SQLN

![Netflix logo](https://github.com/saalimshaikh09-crypto/netflix_sql_project/blob/main/logo.png)

## Objective
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

-- Solutions of 15 business problems

-- 1. Count the number of Movies vs TV Shows
```sql
SELECT 
	type,
	COUNT(*)
FROM netflix
GROUP BY 1
```sql
-- 2. Find the most common rating for movies and TV shows
```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```sql

-- 3. List all movies released in a specific year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020
```sql

-- 4. Find the top 5 countries with the most content on Netflix
```sql
SELECT * 
FROM
(
	SELECT 
		-- country,
		UNNEST(STRING_TO_ARRAY(country, ',')) as country,
		COUNT(*) as total_content
	FROM netflix
	GROUP BY 1
)as t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5
```sql

-- 5. Identify the longest movie
```sql
SELECT 
	*
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
```sql

-- 6. Find content added in the last 5 years
```sql
SELECT
*
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```sql

-- 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
SELECT *
FROM
(

SELECT 
	*,
	UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
FROM 
netflix
)
WHERE 
	director_name = 'Rajiv Chilaka'
```sql


-- 8. List all TV shows with more than 5 seasons
```sql
SELECT *
FROM netflix
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5
```sql

-- 9. Count the number of content items in each genre
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1
```sql

-- 10. Find each year and the average numbers of content release by India on netflix. 
-- return top 5 year with highest avg content release !

```sql
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 
		,2
		)
		as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5
```sql

-- 11. List all movies that are documentaries
```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'
```sql


-- 12. Find all content without a director
```sql
SELECT * FROM netflix
WHERE director IS NULL
```sql

-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```sql

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.


```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```sql

/*
Question 15:
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
*/

```sql
SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1,2
ORDER BY 2
```sql



-- End of reports


