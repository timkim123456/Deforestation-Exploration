# Deforestation-SQL-Project

This is part of the SQL hands on training of Masterschool in partnership with Udacity SQL nanodegree training.

## Introduction
- _The data is obtained data from World Bank comprising forest area and total land area by country, region and year from
1990 to 2016_.
- _The datsets contains details of deforestation in terms of percentage and land change between 1990 - 2016_.

## Aim
- _To explore and analysed the dataset by writing simple SQL queries_.
- _Understand and provide insights with regards to the forestation trend between 1990 - 2016_.


# Analysis and Queries 
```sql
CREATE VIEW forestation AS
SELECT fra.country_code,
        fra.country_name,
        fra.year,
        fra.forest_area_sqkm,
        lda.total_area_sq_mi,
        lda.total_area_sq_mi*2.59 total_area_sqkm,
        reg.region,
        reg.income_group,
        (fra.forest_area_sqkm/(lda.total_area_sq_mi*2.59))*100 AS forest_percent
FROM forest_area AS fra
JOIN land_area AS lda
ON fra.country_code = lda.country_code
AND fra.year = lda.year
JOIN regions reg
ON reg.country_code = lda.country_code
GROUP BY fra.country_code,
    fra.country_name,
    fra.year,
    reg.country_name,
    fra.year,
    reg.income_group,
    reg.region,
    lda.total_area_sq_mi,
    fra.forest_area_sqkm;
```
# GLOBAL SITUATION
a. What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can
use the country record denoted as “World" in the region table.

```sql
SELECT SUM(forest_area_sqkm) AS total_area_of_forest
FROM forestation
WHERE YEAR = 1990
AND country_name = 'World';
```
b. What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can
use the country record in the table is denoted as “World.”

```sql
SELECT SUM(forest_area_sqkm) AS total_area_of_forest
FROM forestation
WHERE YEAR = 2016
AND country_name = 'World';
```
c. What was the change (in sq km) in the forest area of the world from 1990 to 2016?

```sql
SELECT (
      (SELECT SUM(forest_area_sqkm) AS total_forest_area
        FROM forestation
        WHERE YEAR = 1990
        AND country_name = 'World') -
      (SELECT SUM(forest_area_sqkm) AS total_forest_area
FROM forestation
WHERE YEAR = 2016
AND country_name = 'World')) AS Change_in_Forest_Area
FROM Forestation
LIMIT 1;
```
d. What was the percent change in forest area of the world between 1990 and 2016?

```sql
SELECT (((
        (SELECT SUM(forest_area_sqkm) AS total_forest_area
        FROM forestation
        WHERE YEAR = 1990
        AND country_name = 'World')-
        (SELECT SUM(forest_area_sqkm) AS total_forest_area
        FROM forestation
        WHERE YEAR = 2016
        AND country_name = 'World')) / (
        (SELECT SUM(forest_area_sqkm) AS total_forest_area
FROM forestation
WHERE YEAR = 1990
AND country_name='World')))*100) AS
Percent_Change_in_Forest_Area
FROM forestation
LIMIT 1;
```

e. If you compare the amount of forest area lost between 1990 and 2016, to which country's total
area in 2016 is it closest to?
```sql
SELECT country_name,
SUM(total_area_sq_mi*2.59) AS total_area_of_land
FROM forestation
WHERE YEAR = 2016
AND total_area_sq_mi*2.59 <= 1324449
GROUP BY country_name
ORDER BY total_area_of_land DESC
LIMIT 1;
```


# REGIONAL OUTLOOK

2a(i). What was the percent forest of the entire world in 2016?

```sql
SELECT country_name, Round(((SUM(forest_area_sqkm) /
SUM(total_area_sq_mi*2.59))*100)::Numeric, 2) AS Percent_Forest_Area2016
FROM forestation
WHERE YEAR = 2016
AND country_name = 'World'
GROUP BY country_name
```

2a(ii). Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2
decimal places?

```sql
(SELECT region, Round(((SUM(forest_area_sqkm) /
             SUM(total_area_sq_mi*2.59))*100)::Numeric, 2) AS Percent_Forest_Area2016
        FROM forestation
        WHERE YEAR = 2016
        GROUP BY region
        ORDER BY Percent_Forest_Area2016 DESC
        LIMIT 1)
UNION ALL
(SELECT region, Round(((SUM(forest_area_sqkm) /
             SUM(total_area_sq_mi*2.59))*100)::Numeric, 2) AS Percent_Forest_Area2016
        FROM forestation
        WHERE YEAR = 2016
        GROUP BY region
        ORDER BY Percent_Forest_Area2016 ASC
        LIMIT 1);
```

2b(i). What was the percent forest of the entire world in 1990?

```sql
SELECT country_name, Round(((SUM(forest_area_sqkm) /
        SUM(total_area_sq_mi*2.59))*100)::Numeric,2) AS Percent_Forest_Area1990
FROM Forestation
WHERE YEAR = 1990
AND country_name = 'World'
GROUP BY country_name
```

2b(ii). Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2
decimal places?

```sql
(SELECT region, Round(((SUM(forest_area_sqkm) /
          SUM(total_area_sq_mi*2.59))*100)::Numeric, 2) AS Percent_Forest_Area1990
    FROM forestation
    WHERE YEAR = 1990
    GROUP BY region
    ORDER BY Percent_Forest_Area1990 DESC
    LIMIT 1)
UNION ALL
(SELECT region, Round(((SUM(forest_area_sqkm) /
          SUM(total_area_sq_mi*2.59))*100)::Numeric, 2) AS Percent_Forest_Area1990
    FROM forestation
    WHERE YEAR = 1990
    GROUP BY region
    ORDER BY Percent_Forest_Area1990 ASC
    LIMIT 1);
```
2c. Based on the table you created, which regions of the world DECREASED in forest area from
1990 to 2016?

```sql
WITH T1 AS
(SELECT region, Round(((SUM(forest_area_sqkm) /
          SUM(total_area_sq_mi*2.59))*100)::Numeric,2) AS Percent_Forest_Area1990
    FROM forestation
    WHERE YEAR = 1990 GROUP BY region
    ORDER BY Percent_Forest_Area1990 DESC),
T2 AS
(SELECT region, Round(((SUM(forest_area_sqkm) /
          SUM(total_area_sq_mi*2.59))*100)::Numeric,2) AS Percent_Forest_Area2016
      FROM forestation
      WHERE YEAR = 2016 GROUP BY region
      ORDER BY Percent_Forest_Area2016 DESC)
SELECT fra. region, fra. Percent_Forest_Area1990, tra. Percent_Forest_Area2016
FROM T1 AS fra
JOIN T2 AS tra
ON fra. region = tra. region
WHERE fra. Percent_Forest_Area1990>tra. Percent_Forest_Area2016
GROUP BY fra. region, fra. Percent_Forest_Area1990, tra. Percent_Forest_Area2016
LIMIT 2;
```


# COUNTRY-LEVEL DETAIL

3a. Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What
was the difference in forest area for each?

```sql
WITH T1 AS
    (SELECT country_name, Round(SUM(forest_area_sqkm)) AS Forest_Area1990
    FROM forestation
    WHERE YEAR = 1990
    GROUP BY country_name, forest_area_sqkm
    ORDER BY Forest_Area1990 DESC),
T2 AS
    (SELECT country_name, Round(SUM(forest_area_sqkm)) AS Forest_Area2016
    FROM forestation
    WHERE YEAR = 2016
    GROUP BY country_name, forest_area_sqkm
    ORDER BY Forest_Area2016 DESC)
SELECT fra.country_name, fra.Forest_Area1990, tra. Forest_Area2016,
(tra.Forest_Area2016-fra.Forest_Area1990) AS Difference_Land_Area
FROM T1 AS fra
JOIN T2 AS tra
ON fra.country_name = tra.country_name
WHERE fra.country_name != 'World'
GROUP BY fra.country_name, tra.Forest_Area2016, fra.Forest_Area1990
ORDER BY Difference_Land_Area
LIMIT 5;
```


3b. Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What
was the percent change to 2 decimal places for each?

```sql
WITH T1 AS
      (SELECT country_name, region, Round((forest_area_sqkm)::Numeric,2) AS
      Percent_Forest_Area1990
      FROM forestation
      WHERE YEAR = 1990
      GROUP BY country_name, region, forest_area_sqkm),
T2 AS
      (SELECT country_name, region, Round((forest_area_sqkm)::Numeric,2) AS
      Percent_Forest_Area2016
      FROM forestation
      WHERE YEAR = 2016
      GROUP BY country_name, region, forest_area_sqkm)
SELECT fra.country_name, fra.region, fra.Percent_Forest_Area1990,
        tra.Percent_Forest_Area2016, (fra.Percent_Forest_Area1990-
        tra.Percent_Forest_Area2016) AS Difference_Land_Area,
        (((fra.Percent_Forest_Area1990-
        tra.Percent_Forest_Area2016)/fra.Percent_Forest_Area1990)*100) AS
        Difference_Percentage_Land_Area
FROM T1 AS fra
JOIN T2 AS tra
ON fra.country_name = tra.country_name
WHERE fra.Percent_Forest_Area1990 IS NOT NULL
AND tra.Percent_Forest_Area2016 IS NOT NULL
AND fra.country_name != 'World'
GROUP BY fra.country_name, fra.region, fra.Percent_Forest_Area1990, tra.Percent_Forest_Area2016
ORDER BY Difference_Percentage_Land_Area DESC
LIMIT 5;
```


3c. If countries were grouped by percent forestation in quartiles, which group had the most
countries in it in 2016?

```sql
WITH T1 AS
(SELECT country_name, YEAR,
      (SUM(forest_area_sqkm) / SUM(total_area_sq_mi*2.59))*100 AS
      Percent_Forest_in_Quartiles
FROM forestation
WHERE YEAR = 2016
GROUP BY country_name, YEAR, forest_area_sqkm)
SELECT Distinct(quartiles), count(country_name)
        Over(PARTITION BY quartiles)
FROM
      (SELECT country_name,
      CASE
      WHEN Percent_Forest_in_Quartiles <25 THEN '0-25%'
      WHEN Percent_Forest_in_Quartiles >=25
      AND Percent_Forest_in_Quartiles <50 THEN '25-50%'
      WHEN Percent_Forest_in_Quartiles >=50
      AND Percent_Forest_in_Quartiles <75 THEN '50-75%'
      ELSE '75-100%'
      END AS quartiles
FROM T1
WHERE Percent_Forest_in_Quartiles IS NOT NULL
AND YEAR = 2016) sub
```

3d. List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

```sql
WITH T1 AS
(SELECT country_name, YEAR,
      (SUM(forest_area_sqkm) / SUM(total_area_sq_mi*2.59))*100 AS Percent_Forest_in_Quartiles
FROM forestation
WHERE YEAR = 2016
GROUP BY country_name, YEAR, forest_area_sqkm)
SELECT Distinct(quartiles), count(country_name)
        Over(PARTITION BY quartiles)
FROM
(SELECT country_name,
      CASE
      WHEN Percent_Forest_in_Quartiles <25 THEN '0-25%'
      WHEN Percent_Forest_in_Quartiles >=25
      AND Percent_Forest_in_Quartiles <50 THEN '25-50%'
      WHEN Percent_Forest_in_Quartiles >=50
      AND Percent_Forest_in_Quartiles <75 THEN '50-75%'
      ELSE '75-100%'
      END AS quartiles
FROM T1
WHERE Percent_Forest_in_Quartiles IS NOT NULL
AND YEAR = 2016) sub
```

3e. List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

```sql
SELECT country_name, region,forest_percent AS Percent_Forest_in_Quartiles
FROM forestation
WHERE forest_percent > 75 AND year = 2016
GROUP BY country_name, region, forest_percent
ORDER BY Percent_Forest_in_Quartiles DESC;
```

3f. How many countries had a percent forestation higher than the United States in 2016?

```sql
SELECT count(*)
FROM forestation
WHERE forest_percent > (SELECT forest_percent FROM forestation WHERE country_name =
'United States' AND year = 2016) AND year = 2016;
```


# RECOMMENDATIONS
1. Write out a set of recommendations as an analyst on the ForestQuery team. What have you learned from the World Bank data?
This data has extensively shown that the situation around global forestation is not favorable
between 1991 to 2016. Even though if we make consideration at country level, some countries like
China and United States have improved statistics within this period. Hence, it is highly
recommended for countries and regions more affected (Latin America & Caribbean and Sub-
Saharan Africa) to learn more from China and United States. East Asia Pacific region host most
of the world’s countries with the highest forest percentage area with more than 75% forest
designated area. Only two countries from sub-Saharan African (Gabon and Seychelles) are in the
category above 75% of forest designated area.

2. Which countries should we focus on over others?
It is clearly ideal from the results to concentrate all efforts and resources on countries that had the
largest forest area decrease in terms of the (a) percentage decrease and (b) amount decrease.
Based on percentage decrease (Table 3.2) are Togo, Nigeria, Uganda, Mauritania, & Honduras.
Based on amount decrease (Table 3.1) are Brazil, Indonesia, Myanmar, Nigeria, & Tanzania.
Additionally, I am of the opinion that Nigeria should be given a special consideration among the
list above because it has experienced decrease both in terms of percentage and amount of forest
area.