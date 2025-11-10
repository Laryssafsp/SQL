# Funções de janela

## A partida é OVER
A cláusula OVER() permite que você aplique uma função agregada em um conjunto de dados, semelhante às subconsultas em SELECT. A cláusula OVER() oferece benefícios significativos em relação às subconsultas em SELECT, ou seja, suas consultas serão executadas mais rapidamente, e a cláusula OVER() tem uma ampla gama de funções e cláusulas adicionais que você pode incluir, as quais serão abordadas mais adiante neste capítulo.

```sql
SELECT 
	-- Select the id, country name, season, home, and away goals
 	m.id, 
    c.name AS country, 
    m.season,
    m.home_goal,
    m.away_goal,
    -- Use a window to include the aggregate average in each row
	AVG(m.home_goal + m.away_goal) OVER() AS overall_avg
FROM match AS m
LEFT JOIN country AS c ON m.country_id = c.id;
```


## O que você acha do OVER aqui?
As funções de janela permitem que você crie um RANK de informações de acordo com qualquer variável que queira usar para classificar seus dados. Ao configurar isso, você precisará especificar qual coluna/cálculo deseja usar para calcular sua classificação. Isso é feito com a inclusão de uma cláusula
```sql
SELECT 
	-- Select the league name and average goals scored
	l.name AS league,
    AVG(m.home_goal + m.away_goal) AS avg_goals,
    -- Rank each league according to the average goals
    RANK() OVER(ORDER BY AVG(m.home_goal + m.away_goal)) AS league_rank
FROM league AS l
LEFT JOIN match AS m 
ON l.id = m.country_id
WHERE m.season = '2011/2012'
GROUP BY l.name
-- Order the query by the rank you created
ORDER BY league_rank;
```

## Inverta seus resultados com OVER.
No último exercício, a classificação gerada em sua consulta foi organizada da menor para a maior. Ao adicionar DESC à sua função de janela, você pode criar uma classificação ordenada do maior para o menor.
```sql
SELECT 
    -- Select the league name and average goals scored
    l.name AS league,
    AVG(m.home_goal + m.away_goal) AS avg_goals,
    -- Rank leagues in descending order by average goals
    RANK() OVER(ORDER BY AVG(m.home_goal + m.away_goal) DESC) AS league_rank
FROM league AS l
LEFT JOIN match AS m 
ON l.id = m.country_id
WHERE m.season = '2011/2012'
GROUP BY l.name
-- Order the query by the rank you created
ORDER BY league_rank;
```

## PARTITION BY com uma coluna
A cláusula PARTITION BY permite calcular "janelas" separadas com base nas colunas que você deseja dividir os resultados. Por exemplo, você pode criar uma única coluna que calcula a média geral de gols marcados para cada temporada.

Neste exercício, você criará um conjunto de dados de jogos disputados pela Legia Warszawa (Liga de Varsóvia), o time mais bem classificado da Polônia, e comparará o desempenho individual dos jogos com a média geral da temporada.

Onde você vê mais discrepâncias? São jogos do Legia Warszawa em casa ou fora?
```sql
SELECT
    date,
    season,
    home_goal,
    away_goal,
    CASE WHEN hometeam_id = 8673 THEN 'home' 
         ELSE 'away' END AS warsaw_location,
    -- Calculate the average goals scored partitioned by season
    AVG(home_goal) OVER(PARTITION BY season) AS season_homeavg,
    AVG(away_goal) OVER(PARTITION BY season) AS season_awayavg
FROM match
-- Filter the data set for Legia Warszawa matches only
WHERE 
    hometeam_id = 8673 
    OR awayteam_id = 8673
ORDER BY (home_goal + away_goal) DESC;

```

## PARTITION BY com várias colunas
A cláusula PARTITION BY pode ser usada para separar as médias das janelas por vários pontos de dados (colunas). Você pode até calcular as informações que deseja usar para particionar seus dados! Por exemplo, você pode calcular a média de gols marcados por temporada e por país, ou pelo ano (retirado da coluna de data).

Neste exercício, você calculará o número médio de gols marcados em casa e fora pela Legia Warszawa e seus adversários, divididos por mês em cada temporada.
```sql
SELECT 
	date,
    season,
    home_goal,
    away_goal,
    CASE WHEN hometeam_id = 8673 THEN 'home' 
         ELSE 'away' END AS warsaw_location,
    -- Calculate average goals partitioned by season and month
    AVG(home_goal) OVER(PARTITION BY season, 
         	EXTRACT(MONTH FROM date)) AS season_mo_home,
    AVG(away_goal) OVER(PARTITION BY season, 
            EXTRACT(MONTH FROM date)) AS season_mo_away
FROM match
WHERE 
	hometeam_id = 8673 
    OR awayteam_id = 8673
ORDER BY (home_goal + away_goal) DESC;
```

## Deslize para a esquerda
As janelas deslizantes permitem que você crie cálculos acumulados entre dois pontos quaisquer em uma janela usando funções como PRECEDING, FOLLOWING e CURRENT ROW. Você pode calcular contagens em execução, somas, médias e outras funções agregadas entre quaisquer dois pontos que especificar no conjunto de dados.

Neste exercício, você expandirá os exemplos discutidos no vídeo, calculando o total de gols marcados pelo FC Utrecht quando ele era o time anfitrião durante a temporada 2011/2012. Eles marcam mais gols no final da temporada como time anfitrião ou como time visitante?

```sql
SELECT 
    date,
    home_goal,
    away_goal,
    -- Create a running total and running average of home goals
    SUM(home_goal) OVER(ORDER BY date 
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
    AVG(home_goal) OVER(ORDER BY date 
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_avg
FROM match
WHERE 
    hometeam_id = 9908 
    AND season = '2011/2012';
```
### ***ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW***
Essa parte é usada dentro de uma função de janela (OVER(...)) para definir qual intervalo de linhas será considerado no cálculo da função agregada (como SUM, AVG, etc.).

🔍 Quebra da expressão:
- ROWS: indica que estamos lidando com linhas físicas (não valores ou grupos).
- BETWEEN ... AND ...: define o intervalo da janela.
- UNBOUNDED PRECEDING: significa "desde a primeira linha da partição ou conjunto ordenado".
- CURRENT ROW: significa "até a linha atual".

## Deslize para a direita
Agora vamos ver como o FC Utrecht se sai quando é o time visitante. Você vai notar que o total da temporada está na parte inferior do conjunto de dados que você consultou. Dependendo dos seus resultados, isso pode ser bem longo, e rolar para baixo não é muito útil.

Neste exercício, você modificará ligeiramente a consulta do exercício anterior, classificando o conjunto de dados em ordem inversa e calculando um total acumulado de CURRENT ROW até o final do conjunto de dados (registro mais antigo).
```sql
SELECT 
    -- Select the date and away goals
    date,
    away_goal,
    -- Create a running total and running average of away goals
    SUM(away_goal) OVER(ORDER BY date DESC
         ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS running_total,
    AVG(away_goal) OVER(ORDER BY date DESC
         ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS running_avg
FROM match
WHERE 
    awayteam_id = 9908 
    AND season = '2011/2012';

```

## Configuração da CTE to time anfitrião
```sql
SELECT 
    m.id, 
    t.team_long_name,
    -- Identify matches as home/away wins or ties
    CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
         WHEN m.home_goal < m.away_goal THEN 'MU Loss'
         ELSE 'Tie' END AS outcome
FROM match AS m
-- Left join team on the home team ID and team API id
LEFT JOIN team AS t 
ON m.hometeam_id = t.team_api_id
WHERE 
    -- Filter for 2014/2015 and Manchester United as the home team
    m.season = '2014/2015'
    AND t.team_long_name = 'Manchester United'
```

## Configuração do CTE do time visitante

````sql
SELECT 
    m.id, 
    t.team_long_name,
    -- Identify matches as home/away wins or ties
    CASE WHEN m.home_goal > m.away_goal THEN 'MU Loss'
         WHEN m.home_goal < m.away_goal THEN 'MU Win'
         ELSE 'Tie' END AS outcome
-- Join team table to the match table
FROM match AS m
LEFT JOIN team AS t 
ON m.awayteam_id = t.team_api_id
WHERE 
    -- Filter for 2014/2015 and Manchester United as the away team
    m.season = '2014/2015'
    AND t.team_long_name = 'Manchester United';
```
## Juntando as CTEs
````sql
-- Set up the home team CTE
WITH home AS (
  SELECT m.id, t.team_long_name,
      CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
           WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
           ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.hometeam_id = t.team_api_id),
-- Set up the away team CTE
away AS (
  SELECT m.id, t.team_long_name,
      CASE WHEN m.home_goal > m.away_goal THEN 'MU Loss'
           WHEN m.home_goal < m.away_goal THEN 'MU Win' 
           ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.awayteam_id = t.team_api_id)
-- Select team names, the date and goals
SELECT DISTINCT
    m.date,
    home.team_long_name AS home_team,
    away.team_long_name AS away_team,
    m.home_goal,
    m.away_goal
-- Join the CTEs onto the match table
FROM match AS m
LEFT JOIN home ON m.id = home.id
LEFT JOIN away ON m.id = away.id
WHERE m.season = '2014/2015'
      AND (home.team_long_name = 'Manchester United' 
           OR away.team_long_name = 'Manchester United');

```
## Adicionar uma função de janela
````sql
-- Set up the home team CTE
WITH home AS (
  SELECT m.id, t.team_long_name,
      CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
           WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
           ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.hometeam_id = t.team_api_id),
-- Set up the away team CTE
away AS (
  SELECT m.id, t.team_long_name,
      CASE WHEN m.home_goal > m.away_goal THEN 'MU Loss'
           WHEN m.home_goal < m.away_goal THEN 'MU Win' 
           ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.awayteam_id = t.team_api_id)
-- Select columns and and rank the matches by goal difference
SELECT DISTINCT
    m.date,
    home.team_long_name AS home_team,
    away.team_long_name AS away_team,
    m.home_goal, m.away_goal,
    RANK() OVER(ORDER BY ABS(home_goal - away_goal) DESC) as match_rank
-- Join the CTEs onto the match table
FROM match AS m
LEFT JOIN home ON m.id = home.id
LEFT JOIN away ON m.id = away.id
WHERE m.season = '2014/2015'
      AND ((home.team_long_name = 'Manchester United' AND home.outcome = 'MU Loss')
      OR (away.team_long_name = 'Manchester United' AND away.outcome = 'MU Loss'));

```
## 
````sql

```
## 
````sql

```
## 
````sql

```
## 
````sql

```
## 
````sql

```
## 
````sql

```
