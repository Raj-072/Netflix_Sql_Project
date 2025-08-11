# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix logo](logo.png)

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

- **Dataset Link:** [Movies Dataset](netflix_titles.csv)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id	VARCHAR(6),
	type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(208),	
	casts VARCHAR(1000),
	country	VARCHAR(150),
	date_added	VARCHAR(50),
	release_year INT,
	rating	VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)
);

select * from netflix;

SELECT COUNT(*) AS total_content
FROM netflix;

SELECT DISTINCT type
FROM netflix;
```
### 1. Count the number of Movies vs TV Shows

```sql
SELECT type,COUNT(*)
FROM netflix
GROUP BY type;
```
### 2. Find the most common rating for movies and TV shows

```sql
WITH RatingCounts AS (
    SELECT type,rating,COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT type, rating, rating_count,RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT type, rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

### 3. List all movies released in a specific year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE type = 'Movie' AND release_year = 2020;
```

### 4. Find the top 5 countries with the most content on Netflix

```sql
SELECT * 
FROM
(
	SELECT
		UNNEST(STRING_TO_ARRAY(country, ',')) as country,
		COUNT(*) as total_content
	FROM netflix
	GROUP BY 1
)as t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Identify the longest movie

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
```

### 6. Find content added in the last 5 years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

```sql
SELECT *
FROM
(
	SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
	FROM netflix
)
WHERE 
	director_name = 'Rajiv Chilaka';
```


### 8. List all TV shows with more than 5 seasons

```sql
SELECT *
FROM netflix
WHERE TYPE = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

### 9. Count the number of content items in each genre

```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1
```

### 10. Find each year and the average numbers of content release by India on netflix. 
#### return top 5 year with highest avg content release !

```sql
SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
		(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 ,2
		)as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5
```

### 11. List all movies that are documentaries

```sql
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'

```

### 12. Find all content without a director

```sql
SELECT * FROM netflix
WHERE director IS NULL
```

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.


```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### Question 15:
#### Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
#### the description field. Label content containing these keywords as 'Bad' and all other 
##### content as 'Good'. Count how many items fall into each category.

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
```

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
