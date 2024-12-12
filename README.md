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

- **Dataset Link**: [Movies Dataset](#)

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

### 7. List All TV Shows with More than 5 Seasons
```sql
SELECT * 
FROM netflix 
WHERE type = 'TV Show' 
  AND duration IS NOT NULL 
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
**Objective:**  List TV shows with more than 5 seasons.
