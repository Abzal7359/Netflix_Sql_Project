# Netflix Content Analysis Project

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

1. Analyze the distribution of content types (movies vs TV shows).
2. Identify the most common ratings for movies and TV shows.
3. List and analyze content based on release years, countries, and durations.
4. Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link**: [Movies Dataset]('https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download')

### Schema

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
### Business Problems and Solutions
### 1. Count the Number of Movies vs TV Shows
```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```
**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
(SELECT type, rating 
 FROM netflix 
 WHERE type = 'Movie' 
 GROUP BY type, rating 
 ORDER BY COUNT(*) DESC 
 LIMIT 1)

UNION ALL

(SELECT type, rating 
 FROM netflix 
 WHERE type = 'TV Show' 
 GROUP BY type, rating 
 ORDER BY COUNT(*) DESC 
 LIMIT 1);
```
**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
**Objective:** Retrieve all movies released in a specific year.

### 4.Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT 
    UPPER(TRIM(UNNEST(STRING_TO_ARRAY(country, ',')))) AS country, 
    COUNT(*)
FROM netflix 
WHERE country IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC 
LIMIT 5;
```
**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie
```sql
SELECT * 
FROM netflix 
WHERE type = 'Movie' AND duration IS NOT NULL
ORDER BY CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER) DESC 
LIMIT 1;
```
**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years
```sql
SELECT show_id, date_added::DATE 
FROM netflix 
WHERE date_added::DATE >= CURRENT_DATE - INTERVAL '5 years';
```
**Objective:** List content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
WITH movies_TV_shows AS (
    SELECT *, 
           TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director1 
    FROM netflix
)
SELECT * 
FROM movies_TV_shows 
WHERE director1 = 'Rajiv Chilaka';
```
**Objective:** Retrieve all movies and TV shows directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More than 5 Seasons
```sql
SELECT * 
FROM netflix 
WHERE type = 'TV Show' 
  AND duration IS NOT NULL 
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
**Objective:**  List TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre
```sql
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre,
    COUNT(*)  
FROM netflix
GROUP BY 1
ORDER BY 2 DESC;
```
**Objective:** Count the number of content items available in each genre.

### 10. Find Each Year and the Average Percentage of Content Released in India
```sql
WITH country_year_data AS (
    SELECT 
        trim(unnest(string_to_array(country, ','))) AS country, 
        date_part('year', date_added::date) AS year 
    FROM netflix
),
india_count AS (
    SELECT 
        country, 
        COUNT(*) AS india_count 
    FROM country_year_data 
    WHERE country = 'India' 
    GROUP BY country
)
SELECT 
    cyd.country,
    cyd.year,
    COUNT(*) AS year_count,
    ROUND((COUNT(*) * 1.0 / ic.india_count) * 100, 2) AS avg_year 
FROM country_year_data cyd 
LEFT JOIN india_count ic 
ON cyd.country = ic.country
WHERE cyd.country = 'India' 
GROUP BY cyd.country, cyd.year, ic.india_count 
ORDER BY avg_year DESC;
```
**Objective:** Analyze the percentage of content released in India each year.

### 11. List All Movies That Are Documentaries
```sql
SELECT * 
FROM netflix 
WHERE listed_in LIKE '%Documentaries%';
```
**Objective:** Identify all Netflix movies categorized as documentaries.

### 12. Find All Content Without a Director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
**Objective:** List all content items that do not have a specified director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in During the Last 10 Years
```sql
SELECT * 
FROM netflix 
WHERE LOWER(casts) LIKE '%salman khan%' 
AND release_year >= date_part('year', CURRENT_DATE) - 10;
```
**Objective:** Determine the number of movies featuring Salman Khan released in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
SELECT 
    trim(unnest(string_to_array(casts, ','))) AS actors,
    COUNT(*) 
FROM netflix 
WHERE country = 'India'
GROUP BY actors 
ORDER BY COUNT(*) DESC
LIMIT 10;
```
**Objective:** Identify the actors who have featured in the most movies produced in India.

### 15. Categorize Content Based on the Presence of the Keywords 'Kill' and 'Violence'
```sql
SELECT content_type, COUNT(*)
FROM (
    SELECT *,
           (CASE
               WHEN LOWER(description) LIKE '% kill%' OR LOWER(description) LIKE '% violence%' THEN 'Bad'
               ELSE 'Good'
           END) AS content_type
    FROM netflix
) AS content_group
GROUP BY content_type;
```
**Objective:** Categorize Netflix content into 'Good' or 'Bad' based on the presence of specific keywords.
