# NetflixDataAnalysisSQL
![NETFLIXL Logo](https://github.com/d-image/NetflixDataAnalysisSQL/blob/main/NetflixLogo.png)
End to end analysis with Netflix data in SQL

## Overview
This project requires a thorough examination of Netflix's movie and TV show data using SQL. The purpose is to extract relevant insights and answer a variety of (20+) business questions using the dataset. This paper details the project's objectives, business challenges, solutions, findings, and conclusions.
 
## Objectives
 
- Examine the distribution of content types (movies versus television shows).
- Determine the most frequent ratings for films and television shows.
- Organise and analyse content by release year, country, and duration.
- Analyse and categorise information using precise criteria and keywords.
 
## Dataset
 
Though the dataset for this project is sourced from the Kaggle dataset, but its uploaded here: Netflix_titles.csv
 
 
## Business Problems and Solutions
 
### 1. Display the total Number of Movies vs TV Shows
 
```sql
SELECT 
   type,
   COUNT(*) count_type
FROM netflix_titles
GROUP BY type
```
**Objective:** Determine the distribution of content types on Netflix.

### 2.	Count the Number of Content Items in Each Genre

```sql
   SELECT 
	Trim(Value) AS genre,  
	COUNT(*) AS total_content  
FROM netflix_titles
   CROSS APPLY string_split (listed_in, ',') 
GROUP BY Trim(Value);
```
### Objective: Count the number of content items in each genre

### 3.	List All Movies Released in a 2020
```sql
    SELECT * 
    FROM netflix_titles
   WHERE release_year = 2020;
 ```

### Objective: Retrieve all movies released in a specific year.

### 4.	Find the Top 5 Countries with the Most Content on Netflix
```sql
    SELECT Top(5) * 
		FROM
		(
			SELECT 
			Trim(Value) AS country,  
			COUNT(*) AS total_content  
			FROM netflix_titles
			   CROSS APPLY string_split (country, ',') 
			GROUP BY Trim(Value)

		) AS temp
		WHERE country IS NOT NULL
		ORDER BY total_content DESC
```

### Objective: Identify the top 5 countries with the highest number of content items.


 
### 5.	Find Content Added in the Last 5 Years

```sql
   SELECT 
	* 
FROM 
	netflix_titles_filter
WHERE 
	date_added >= DATEADD(Year, -5, GetDate())
``` 
 
 
### 6.	List All Movies that are Documentaries

--Method 1

```sql
    SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND Listed_in LIKE '%Documentaries%'
``` 
 
--Method 2

```sql
   SELECT ntf.*, nli.listed_in 

    FROM netflix_titles_filter ntf

    JOIN netflix_listed_in nli

    ON ntf.show_id = nli.show_id

    WHERE nli.Listed_in = 'Documentaries'
 
    SELECT * FROM netflix_listed_in
``` 
 
 
### 7.	Find All Content Without a Director
 
```sql
SELECT * FROM netflix_titles_filter

WHERE director = 'NA'
``` 
 
--Method 2

```sql
SELECT ntf.*, nd.director 

FROM netflix_titles_filter ntf

JOIN netflix_director nd

ON ntf.show_id = nd.show_id

WHERE nd.director = 'NA'
```

### 8.	Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

--Method 1

```sql
SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND cast LIKE '%Salman Khan%' AND  release_year > YEAR(GetDate()) - 10
``` 
--Method 2

```sql
SELECT ntf.*, nc.cast 

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE nc.cast = 'Salman Khan' AND  ntf.release_year > YEAR(GetDate()) - 10
``` 
 
 
### 9.	Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

--Method 1

```sql
SELECT TOP (10)

	Trim(Value) AS Actor,

	COUNT(*) HighestNumber

FROM netflix_titles_filter

CROSS APPLY STRING_SPLIT(cast,',')

WHERE country LIKE '%India%' AND type = 'Movie'

GROUP BY Trim(Value)

Order BY COUNT(*) DESC
``` 
--Method 2, Using JOIN

```sql
SELECT TOP (10) trim(cast) Actor, Count(*) HighestNumber

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE ntf.country = 'India' AND ntf.type = 'Movie'

GROUP BY trim(cast)

Order BY COUNT(*) DESC
``` 
 
### 10.	Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
 
--Method 1

```sql
SELECT Category, Count(*) CategoryCounts
FROM
	(
	SELECT description,
		CASE	
			WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
		ELSE 
			'Good'
		END AS Category
	FROM Netflix_Titles_Filter
	)AS CategorizedContents
GROUP BY Category
```
--Method 2

```sql
SELECT 
	CASE
		WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
		ELSE 'Good'
	END as Category,
	Count(*) as TotalCount
FROM Netflix_Titles_Filter
GROUP BY 
	CASE
		WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
		ELSE 'Good'
	END;
```
 
 ### 11.	Identify the Longest Movie

-- Our strint_split has another parameter called the ORDINAL VALUE. That when you use string_split, it will split the value of the column and return the first value as Ordinal 1, means the first value and not the second value. ORDINAL is like the index of the numbering. We CAST the Value because we need the answer as INTEGER.
 
```sql
SELECT TOP 1
	Type,
	Title,
	Trim(Value) as TotalMunite,
	Duration
FROM Netflix_Titles_Filter
CROSS APPLY string_split(duration, ' ', 1)
WHERE type = 'Movie' AND ORDINAL = 1
ORDER BY CAST(Trim(Value) AS INT) DESC
``` 
 
### 12.	Find All Movies/TV Shows by Director 'Rajiv Chilaka'

--Method 1

```sql
SELECT * FROM netflix_titles_filter
WHERE Type IN ('Movie', 'TV Show') AND Director LIKE '%Rajiv Chilaka%'
``` 
--Method 2

```sql
SELECT * FROM netflix_titles_filter
WHERE Director LIKE '%Rajiv Chilaka%'
 ```
--Method 3

```sql
SELECT *, ntf.type, nd.director 
FROM netflix_titles_filter ntf
JOIN netflix_director nd
ON ntf.show_id = nd.show_id
WHERE ntf.Type = 'Movie' AND nd.Director = 'Rajiv Chilaka'
```
