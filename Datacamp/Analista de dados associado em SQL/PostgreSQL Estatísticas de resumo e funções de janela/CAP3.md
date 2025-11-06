## Totais de medalhas de atletas
O total em execução (ou soma cumulativa) de uma coluna ajuda você a determinar qual é a contribuição de cada linha para a soma total.

```sql
WITH Athlete_Medals AS (
  SELECT
    Athlete, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country = 'USA' AND Medal = 'Gold'
    AND Year >= 2000
  GROUP BY Athlete)

SELECT
  -- Calculate the running total of athlete medals
  Athlete,
  Medals,
  SUM(Medals) OVER (ORDER BY Athlete ASC) AS Max_Medals
FROM Athlete_Medals
ORDER BY Athlete ASC;
```
## Máximo de medalhas por país por ano
Obter o máximo de medalhas conquistadas por um país até o momento ajuda você a determinar se um país quebrou seu recorde de medalhas, comparando as medalhas conquistadas no ano atual e o máximo até o momento.

```sql
WITH Country_Medals AS (
  SELECT
    Year, Country, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country IN ('CHN', 'KOR', 'JPN')
    AND Medal = 'Gold' AND Year >= 2000
  GROUP BY Year, Country)

SELECT
  -- Return the max medals earned so far per country
  Year,
  Country,
  Medals,
  SUM(Medals) OVER (PARTITION BY Country
                    ORDER BY Year ASC) AS Max_Medals
FROM Country_Medals
ORDER BY Country ASC, Year ASC;
```
## Mínimo de medalhas por país por ano
Até agora, você viu MAX e SUM, funções agregadas normalmente usadas com GROUP BY, sendo usadas como funções de janela. Você também pode usar as outras funções agregadas, como MIN, como funções de janela.

```sql
WITH France_Medals AS (
  SELECT
    Year, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country = 'FRA'
    AND Medal = 'Gold' AND Year >= 2000
  GROUP BY Year)

SELECT
  Year,
  Medals,
  MIN(Medals) OVER (ORDER BY Year ASC) AS Min_Medals
FROM France_Medals
ORDER BY Year ASC;
```
## Máximo de movimentação das medalhas dos atletas escandinavos
Os quadros permitem que você restrinja as linhas passadas como entrada para a sua função de janela a uma janela deslizante para que você defina o início e o fim.
Ao adicionar um quadro à sua função de janela, você pode calcular métricas "móveis", cujas entradas deslizam de uma linha para outra.

```sql
WITH Scandinavian_Medals AS (
  SELECT
    Year, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country IN ('DEN', 'NOR', 'FIN', 'SWE', 'ISL')
    AND Medal = 'Gold'
  GROUP BY Year)

SELECT
  -- Select each year's medals
  Year,
  Medals,
  -- Get the max of the current and next years'  medals
  MAX(Medals) OVER (ORDER BY Year ASC
             ROWS BETWEEN CURRENT ROW
             AND 1 FOLLOWING) AS Max_Medals
FROM Scandinavian_Medals
ORDER BY Year ASC;
```
## Movimentação máxima das medalhas dos atletas chineses
Os quadros permitem que você "espie" para frente ou para trás sem primeiro usar as funções de busca relativa, LAG e LEAD, para buscar os valores das linhas anteriores na linha atual.

```sql
WITH Chinese_Medals AS (
  SELECT
    Athlete, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country = 'CHN' AND Medal = 'Gold'
    AND Year >= 2000
  GROUP BY Athlete)

SELECT
  -- Select the athletes and the medals they've earned
  Athlete,
  Medals,
  -- Get the max of the last two and current rows' medals 
  MAX(Medals) OVER (ORDER BY Athlete ASC
              ROWS BETWEEN 2 PRECEDING
              AND CURRENT ROW) AS Max_Medals
FROM Chinese_Medals
ORDER BY Athlete ASC;
```
## Média móvel de medalhas russas
O uso de quadros com funções de janela agregadas permite que você calcule muitas métricas comuns, incluindo médias móveis e totais. 
Essas métricas acompanham a mudança no desempenho ao longo do tempo.

```sql
WITH Russian_Medals AS (
  SELECT
    Year, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country = 'RUS'
    AND Medal = 'Gold'
    AND Year >= 1980
  GROUP BY Year)

SELECT
  Year, Medals,
  --- Calculate the 3-year moving average of medals earned
   AVG(Medals)  OVER
    (ORDER BY Year ASC
     ROWS BETWEEN
     2 PRECEDING AND CURRENT ROW) AS Medals_MA
FROM Russian_Medals
ORDER BY Year ASC;
```
## Total móvel de medalhas dos países
E se os seus dados estiverem divididos em vários grupos espalhados em uma ou mais colunas da tabela? Mesmo com um quadro definido, 
se você não conseguir separar de alguma forma os dados dos grupos, os valores de um grupo afetarão a média dos valores de outro grupo.

```sql
WITH Country_Medals AS (
  SELECT
    Year, Country, COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Year, Country)

SELECT
  Year, Country, Medals,
  -- Calculate each country's 3-game moving total
  SUM(Medals) OVER
    (PARTITION BY Country
     ORDER BY Year ASC
     ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS Medals_MA
FROM Country_Medals
ORDER BY Country ASC, Year ASC;
```
