# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/Mayank0141/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL(MySQL). The goal is to extract valuable insights and answer various business questions based on the dataset. 

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Business Problems & Solutions

### 1. count of movies and tv shows
```sql
select type, count(type) from netflix.netflix
group by type;
```

### 2. most common rating for movies and tvshows
```sql
select type, rating from
(select type, rating, count(*), rank() over(partition by type order by count(*) desc) as ranking from netflix
group by 1,2) as t1
 where ranking = 1;
``` 

 ### 3. all movies released in a specific year e.g, 2020
 ```sql
 select show_id from netflix
 where type = 'Movie' and release_year = '2020';
``` 

 ### 4. top 5 countries with most content on netflix
 ```sql
 WITH SplitCountries AS (SELECT show_id, TRIM(JSON_UNQUOTE(JSON_EXTRACT(countries.value, '$'))) AS country 
 FROM netflix, JSON_TABLE(CONCAT('["', REPLACE(country, ', ', '","'), '"]'), '$[*]' COLUMNS(value JSON PATH '$'))
 AS countries)
SELECT country, COUNT(*) AS content_count FROM SplitCountries
WHERE country IS NOT NULL
GROUP BY country
ORDER BY content_count DESC
LIMIT 5;
```

### 5. identify the longest movie
```sql
SELECT title, duration FROM netflix
WHERE type = 'Movie'
ORDER BY CAST(SUBSTRING_INDEX(duration, ' ', 1) as unsigned) DESC
limit 1;
```

### 6. Find content added in the last 5 years
```sql
SELECT title, type, date_added FROM netflix
WHERE 3 IS NOT NULL AND 
STR_TO_DATE(date_added, '%M %d, %Y') >= DATE_SUB((SELECT MAX(STR_TO_DATE(date_added, '%M %d, %Y')) FROM netflix), 
INTERVAL 5 YEAR)
ORDER BY 3 DESC;

SELECT title, type, date_added FROM netflix
WHERE 3 IS NOT NULL AND STR_TO_DATE(date_added, '%M %d, %Y') >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR)
ORDER BY 3 DESC;
```

### 7. all the movies/tv shows by director 'rajiv chilaka'
```sql
select title, type, director from netflix
where director like '%rajiv chilaka%';
```

### 8. tvshows with more than 5 seasons
```sql
select title, duration from netflix
where type = 'TV Show' and CAST(SUBSTRING_INDEX(duration, ' ', 1) as unsigned) > 5;
```

### 9. no. of content items in each genre
```sql
WITH Splitgenre AS (SELECT show_id, TRIM(JSON_UNQUOTE(JSON_EXTRACT(genres.value, '$'))) AS genre 
 FROM netflix, JSON_TABLE(CONCAT('["', REPLACE(listed_in, ', ', '","'), '"]'), '$[*]' COLUMNS(value JSON PATH '$'))
 AS genres)
SELECT genre, COUNT(*) AS content_count FROM Splitgenre
WHERE genre IS NOT NULL
GROUP BY genre
ORDER BY content_count DESC;
```

### 10. each year and avg no. of content release by india on netflix, return top 5 year with highest avg content release.
```sql
select year(STR_TO_DATE(date_added, '%M %d, %Y')) as year_added, count(*),
(count(*)/(select count(*) from netflix where country like '%India%') * 100) as avg_content from netflix
where country like '%India%'
group by year_added
order by avg_content desc;
```

### 11. list all movies that are documentaries
```sql
select title, listed_in from netflix
where type = 'Movie' and listed_in like '%Documentaries%';
```

### 12. all content without director
```sql
select title from netflix
where director is null;
```

### 13. find how many movies actor 'salman khan' appeared in last 10 years
```sql
SELECT COUNT(*) AS movie_count FROM netflix
WHERE type = 'Movie' AND cast LIKE '%Salman Khan%' AND release_year >= YEAR(CURDATE()) - 10;
```

### 14. top 10 actors who have appeared in the highest no. of movies produced in India
```sql
WITH Splitactors AS (SELECT show_id, TRIM(JSON_UNQUOTE(JSON_EXTRACT(actors.value, '$'))) AS actor 
 FROM netflix, JSON_TABLE(CONCAT('["', REPLACE(cast, ', ', '","'), '"]'), '$[*]' COLUMNS(value JSON PATH '$')) AS actors
 where country like '%India%' and type = 'Movie')
SELECT actor, COUNT(*) AS content_count FROM Splitactors
WHERE actor IS NOT NULL
GROUP BY actor
ORDER BY content_count DESC
limit 10;
```

### 15. categorize content based on the presence of words 'kill' and 'violence' in the description field, label content content having these words 'bad' and all other content as 'good'. count how many fall into each category
```sql
SELECT CASE WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad-Content' ELSE 'Good_Content'
END AS content_category, COUNT(*) AS content_count FROM netflix
WHERE description IS NOT NULL
GROUP BY content_category;
```
