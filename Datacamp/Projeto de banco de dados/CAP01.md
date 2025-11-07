## Decisão de tabelas de fatos e dimensões
Imagine que você adora dados e correr. É natural que você comece a coletar dados sobre sua rotina semanal de corrida. 
Sua maior preocupação é monitorar quanto tempo está correndo a cada semana. Você também registra a rota e as distâncias de suas corridas.

```sql
-- Create a route dimension table
CREATE TABLE route (
	route_id INTEGER PRIMARY KEY,
    park_name VARCHAR(160) NOT NULL,
    city_name VARCHAR(160) NOT NULL,
    distance_km FLOAT NOT NULL,
    route_name VARCHAR(160) NOT NULL
);
-- Create a week dimension table
CREATE TABLE week(
	week_id INTEGER PRIMARY KEY,
    week INTEGER NOT NULL,
    month VARCHAR(160) NOT NULL,
    year INTEGER NOT NULL
);
```
## Consultando o modelo dimensional

```sql
SELECT 
	-- Get the total duration of all runs
	SUM(duration_mins)
FROM 
	runs_fact
-- Get all the week_id's that are from July, 2019
INNER JOIN week_dim ON week_dim.week_id = runs_fact.week_id
WHERE month = 'July' and year = '2019';
```
