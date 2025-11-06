## Adicionar uma subconsulta à cláusula SELECT
As subconsultas nos comandos SELECT geram um único valor que permite passar um valor agregado para um data frame. Isso é útil para realizar cálculos nos dados do seu banco de dados.
```SQL
SELECT 
    l.name AS league,
    -- Select and round the league's total goals
    ROUND(AVG(m.home_goal + m.away_goal), 2) AS avg_goals,
    -- Select & round the average total goals for the season
    (SELECT ROUND(AVG(home_goal + away_goal), 2)
     FROM match
     WHERE season = '2013/2014') AS overall_avg
FROM league AS l
LEFT JOIN match AS m
ON l.country_id = m.country_id
-- Filter for the 2013/2014 season
WHERE m.season = '2013/2014'
GROUP BY l.name;
```
## Subconsultas no Select para cálculos
As subconsultas em SELECT são uma maneira útil de criar colunas calculadas em uma consulta. Uma subconsulta em SELECT pode ser tratada como um único valor numérico a ser usado em seus cálculos. Ao escrever consultas em SELECT, é importante lembrar que a filtragem da consulta principal não filtra a subconsulta e vice-versa.
```SQL
SELECT
    -- Select the league name and average goals scored
    l.name AS league,
    ROUND(AVG(m.home_goal + m.away_goal), 2) AS avg_goals,
    
    -- Subtract the overall average from the league average
    ROUND(
        AVG(m.home_goal + m.away_goal) 
        - 
        (SELECT AVG(home_goal + away_goal)
         FROM match
         WHERE season = '2013/2014'),
    2) AS diff

FROM league AS l
LEFT JOIN match AS m
ON l.country_id = m.country_id

-- Only include 2013/2014 results
WHERE m.season = '2013/2014'

GROUP BY l.name;
```
