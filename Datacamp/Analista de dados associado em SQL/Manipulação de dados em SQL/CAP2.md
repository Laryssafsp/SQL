##Filtragem usando subconsultas escalares
As subconsultas são incrivelmente eficientes para a realização de filtros e transformações complexas. Você pode filtrar dados com base em valores únicos (escalares) usando uma subconsulta de uma forma que não é possível usando instruções WHERE ou uniões. As subconsultas também podem ser usadas para manipulação mais avançada de seu conjunto de dados. Você provavelmente encontrará subconsultas em qualquer ambiente do mundo real que use bancos de dados relacionais.

Neste exercício, você gerará uma lista de partidas em que o total de gols marcados (totalizando os dois times) é mais de 3 vezes a média dos jogos na tabela matches_2013_2014, que inclui todos os jogos disputados na temporada 2013/2014.

```SQL
SELECT 
    -- Select the date, home goals, and away goals scored
    date,
    home_goal,
    away_goal
FROM matches_2013_2014
-- Filter for matches where total goals exceeds 3x the average
WHERE (home_goal + away_goal) > 
      (SELECT 3 * AVG(home_goal + away_goal)
       FROM matches_2013_2014);

```
##Filtragem usando uma subconsulta com uma lista
Seu objetivo neste exercício é gerar uma lista de times que nunca jogaram uma partida em casa. Usando uma subconsulta, você gerará uma lista de valores únicos de hometeam_ID da tabela match não filtrada para excluir na coluna team_api_ID da tabela team.

Além de filtrar usando uma subconsulta de valor único (escalar), você pode criar uma lista de valores em uma subconsulta para filtrar dados com base em um conjunto complexo de condições. Esse tipo de subconsulta gera uma lista de referência de uma coluna para a consulta principal. Desde que os valores da sua lista correspondam a uma coluna na tabela da consulta principal, você não precisa usar uma união, mesmo que a lista seja de uma tabela separada.

```SQL
SELECT 
    -- Select the team long and short names
    team_long_name,
    team_short_name
FROM team
-- Exclude all values from the subquery
WHERE team_api_id NOT IN
      (SELECT DISTINCT hometeam_ID FROM match);
```
## Filtragem com condições de subconsulta mais complexas
No exercício anterior, você gerou uma lista de times que não têm jogos em casa listados no banco de dados de futebol usando uma subconsulta em WHERE. Vamos explorar um pouco mais esse banco de dados, criando uma lista de times que marcaram 8 ou mais gols em uma partida em casa.

Para fazer isso, você construirá uma subconsulta na cláusula WHERE com sua própria condição de filtragem.
```SQL
SELECT
	-- Select the team long and short names
    team_long_name,
    team_short_name
FROM team
-- Filter for teams with 8 or more home goals
WHERE team_api_id IN (
    SELECT hometeam_id
    FROM match
    WHERE home_goal >= 8);
```
## Unindo subconsultas em FROM
A tabela match no European Soccer Database não contém nomes de países ou times. Você pode obter essas informações unindo-as à tabela country e usá-las para agregar informações, como o número de partidas disputadas em cada país.

Se você estiver interessado em filtrar dados de uma dessas tabelas, também poderá criar uma subconsulta de uma das tabelas e, em seguida, uni-la a uma tabela existente no banco de dados. Uma subconsulta em FROM é uma maneira eficaz de responder a perguntas detalhadas que exigem filtragem ou transformação de dados antes de incluí-los nos resultados finais.

Seu objetivo neste exercício é gerar uma subconsulta usando a tabela match e, em seguida, unir essa subconsulta à tabela country para calcular informações sobre partidas com 10 ou mais gols no total.

```SQL
SELECT
	-- Select country name and the count match IDs
    c.name AS country_name,
    COUNT(sub.id) AS matches
FROM country AS c
-- Inner join the subquery onto country
-- Select the country id and match id columns
INNER JOIN (SELECT country_id, id 
            FROM match
            -- Filter the subquery by matches with 10+ goals
            WHERE (home_goal + away_goal) >= 10) AS sub
ON c.id = sub.country_id
GROUP BY country_name;
```
## Baseando-se em subconsultas em FROM
No exercício anterior, você descobriu que Inglaterra, Holanda, Alemanha e Espanha foram os únicos países no banco de dados que tiveram partidas com 10 ou mais gols no total. 
Vamos descobrir mais alguns detalhes sobre essas partidas: quando foram disputadas, em quais temporadas e quantos gols foram marcados em casa ou fora.

Você perceberá que, neste exercício, o alias da tabela é excluído para cada coluna selecionada na consulta principal.
Isso ocorre porque a consulta principal está extraindo dados da subconsulta, que é tratada como uma única tabela.
```SQL
SELECT
    -- Select country, date, home, and away goals from the subquery
    country,
    date,
    home_goal,
    away_goal
FROM 
    -- Select country name, date, home_goal, away_goal, and total goals in the subquery
    (SELECT c.name AS country, 
            m.date, 
            m.home_goal, 
            m.away_goal,
            (m.home_goal + m.away_goal) AS total_goals
     FROM match AS m
     LEFT JOIN country AS c
     ON m.country_id = c.id) AS subq
-- Filter by total goals scored in the main query
WHERE total_goals >= 10;
```
