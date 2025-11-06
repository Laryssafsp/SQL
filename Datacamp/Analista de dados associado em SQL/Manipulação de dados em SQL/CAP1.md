## Comandos CASE básicos
Qual é o seu time favorito?

O banco European Soccer Database contém dados sobre 12.800 partidas de 11 países disputadas entre 2011 e 2015. Ao longo deste curso, você verá versões filtradas das tabelas desse banco de dados para explorar melhor seu conteúdo.

Neste exercício, você identificará as partidas disputadas entre o FC Schalke 04 e o FC Bayern de Munique. Há dois times identificados em cada partida nas colunas hometeam_id e awayteam_id da tabela filtrada matches_germany. Você pode unir o ID à coluna team_api_id na tabela teams_germany, mas não pode executar uma união com as duas ao mesmo tempo.

No entanto, você pode realizar essa operação usando uma declaração CASE depois de identificar o team_api_id associado a cada time.

```sql
-- Identify the home team as Bayern Munich, Schalke 04, or neither
SELECT 
	CASE WHEN hometeam_id = 10189 THEN 'FC Schalke 04'
        WHEN hometeam_id = 9823 THEN 'FC Bayern Munich'
         ELSE 'Other' END AS home_team,
	COUNT(id) AS total_matches
FROM matches_germany
GROUP BY home_team;
```

## Comandos CASE comparando valores de colunas
O Barcelona é considerado um dos times mais fortes da liga de futebol da Espanha.

Neste exercício, você criará uma lista de partidas da temporada 2011/2012 em que o Barcelona foi o time anfitrião. Você fará isso usando um comando CASE que compara os valores de duas colunas para criar um novo grupo: vitórias, derrotas e empates.

Em três etapas, você criará uma consulta que identifica o vencedor de uma partida, identifica o adversário e, por fim, filtra o Barcelona como o time anfitrião. Ao concluir uma consulta nessa ordem, você poderá observar os resultados tomando forma a cada nova informação.

A tabela matches_spain contém atualmente os jogos do Barcelona da temporada 2011/2012 e tem duas colunas-chave, hometeam_id e awayteam_id, que podem ser unidas à tabela teams_spain. No entanto, você só pode unir teams_spain a uma coluna de cada vez.
```SQL
SELECT 
	date,
	-- Identify home wins, losses, or ties
	CASE WHEN home_goal > away_goal THEN 'Home win!'
        WHEN home_goal < away_goal THEN 'Home loss :(' 
        ELSE 'Tie' END outcome
FROM matches_spain;
```

## Comandos CASE que comparam valores de duas coluna, parte 2
Semelhante ao exercício anterior, você criará uma consulta para determinar o resultado das partidas do Barcelona em que ele jogou como time visitante. Você aprenderá a combinar essas duas consultas nos capítulos 2 e 3.

O desempenho deles foi diferente das partidas em que eram o time anfitrião?
```SQL
-- Select matches where Barcelona was the away team
SELECT
	m.date,
	t.team_long_name AS opponent,
	CASE WHEN m.home_goal < m.away_goal THEN 'Barcelona win!'
         WHEN m.home_goal > m.away_goal THEN 'Barcelona loss :('
         ELSE 'Tie' END AS outcome
FROM matches_spain AS m
LEFT JOIN teams_spain AS t
ON m.hometeam_id = t.team_api_id
-- Filter for Barcelona
WHERE m.awayteam_id = 8634;
```

## Em CASE de rivalidade
O Barcelona e o Real Madrid são times rivais há mais de 80 anos. As partidas entre esses dois times recebem o nome de El Clásico (O Clássico). Neste exercício, você consultará uma lista de partidas disputadas entre esses dois rivais, em que o Barcelona é o time da casa, 
categorizando como uma vitória em casa ou fora, dependendo de várias condições.
```SQL
SELECT 
	date,
	CASE WHEN hometeam_id = 8634 THEN 'FC Barcelona' 
         ELSE 'Real Madrid CF' END as home,
	CASE WHEN awayteam_id = 8634 THEN 'FC Barcelona' 
         ELSE 'Real Madrid CF' END as away,
	-- Identify possible home match outcomes
	CASE WHEN home_goal > away_goal AND hometeam_id = 8634 THEN 'Barcelona win!'
        WHEN home_goal < away_goal AND awayteam_id = 8633 THEN 'Real Madrid win!'
        ELSE 'Tie!' END as outcome
FROM matches_spain
WHERE hometeam_id = 8634 AND awayteam_id = 8633;
```

## Filtrando seu comando CASE
Vamos gerar uma lista de partidas vencidas pelo Bologna da Itália! Há vários outros times nas duas tabelas, 
portanto, para gerar uma consulta que traga o resultado que você quer, 
é preciso usar a declaração `CASE` como um filtro na cláusula WHERE.

Os comandos `CASE` permitem que você categorize os dados nos quais está interessado e exclua os que não interessam. 
Para fazer isso, você pode usar um comando `CASE` como um filtro na cláusula `WHERE` para retornar apenas o que deseja ver.

Veja como você pode configurar isso:
```
SELECT *
FROM table
WHERE 
    CASE WHEN a > 5 THEN 'Keep'
         WHEN a <= 5 THEN 'Exclude' END = 'Keep';
```
Basicamente, você pode usar o comando `CASE` como uma coluna de filtragem, como qualquer outra coluna no seu banco de dados. 
A única diferença é que você não usa o alias do comando na cláusula `WHERE`.

```SQL
SELECT 
	season,
	date,
	home_goal,
	away_goal
FROM matches_italy
WHERE
	-- Find games where home_goal is more than away_goal
	CASE WHEN hometeam_id = 9857 AND home_goal > away_goal THEN 'Bologna Win'
    	-- Find games where away_goal is more than home_goal
        WHEN awayteam_id = 9857 AND away_goal > home_goal THEN 'Bologna Win'
        -- Exclude games not won by Bologna
        END IS NOT NULL;
```
## COUNT usando CASE WHEN
O número de partidas de futebol disputadas em um determinado país europeu difere entre as temporadas? Usaremos o European Soccer Database para responder a essa pergunta.

Você examinará o número de partidas disputadas em 3 temporadas em cada país listado no banco de dados. 
Isso é muito mais fácil de explorar com as partidas de cada temporada em colunas separadas. Usando a tabela country e não filtrada match, 
você contará o número de partidas disputadas em cada país durante as temporadas 2012/2013 e 2013/2014.

```SQL
SELECT 
    c.name AS country,
    -- Count matches in 2012/13
    COUNT(CASE WHEN m.season = '2012/2013' THEN m.id END) AS matches_2012_2013,
    -- Count matches in 2013/14
    COUNT(CASE WHEN m.season = '2013/2014' THEN m.id END) AS matches_2013_2014
FROM country AS c
LEFT JOIN match AS m
ON c.id = m.country_id
GROUP BY country;
```

## Filtragem e totalização usando CASE WHEN
Você pode usar os comandos CASE para aplicar um filtro e realizar um cálculo, escrevendo o comando dentro de uma função de agregação, 
como SUM()!
Neste exercício, seu objetivo é filtrar um time específico (Real Sociedad) e calcular o total de gols em casa e fora por temporada.

```SQL
SELECT season,
    -- SUM the home goals
    SUM(CASE WHEN hometeam_id = 8560 THEN home_goal END) AS home_goals,
    -- SUM the away goals
    SUM(CASE WHEN awayteam_id = 8560 THEN away_goal END) AS away_goals
FROM match
-- Group the results by season
GROUP BY season;
```

## Cálculo de porcentagem com CASE e AVG

Os comandos `CASE` retornam qualquer valor que você especificar na cláusula `THEN`.  
Essa é uma ferramenta extremamente poderosa para cálculos robustos e manipulação de dados quando usada em conjunto com funções de agregação.

Uma das tarefas mais úteis é usar `CASE` dentro de uma função `AVG` para calcular porcentagens no banco de dados.  
Isso funciona porque a média de 1s e 0s é exatamente a proporção de registros que atendem a determinada condição.

### Exemplo:

```sql
AVG(CASE 
        WHEN condition_is_met THEN 1
        WHEN condition_is_not_met THEN 0 
    END)
```
Com essa abordagem, é fundamental garantir que todos os registros sejam convertidos em 1 ou 0 — caso contrário, 
o cálculo da porcentagem pode se tornar incorreto.

Calcular a porcentagem de empates para cada país, em duas temporadas diferentes (2013/2014 e 2014/2015).

```SQL
SELECT 
	c.name AS country,
    -- Calculate the percentage of tied games in each season
	AVG(CASE WHEN m.season= '2013/2014' AND m.home_goal = m.away_goal THEN 1
			 WHEN m.season= '2013/2014' AND m.home_goal != m.away_goal THEN 0
			 END) AS ties_2013_2014,
	AVG(CASE WHEN m.season= '2014/2015' AND m.home_goal = m.away_goal THEN 1
			 WHEN m.season= '2014/2015' AND m.home_goal != m.away_goal THEN 0
			 END) AS ties_2014_2015
FROM country AS c
LEFT JOIN matches AS m
ON c.id = m.country_id
GROUP BY country;
```
