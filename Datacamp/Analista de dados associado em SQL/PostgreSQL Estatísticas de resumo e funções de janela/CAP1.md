## Numeração de linhas
O aplicativo mais simples para as funções de janela é a numeração de linhas. 
A numeração das linhas permite que você busque facilmente a nª linha. 
Por exemplo, seria muito difícil obter a 35ª linha em uma determinada tabela se você não tivesse uma coluna com o número de cada linha. 

```sql
SELECT
  *,
  -- Assign numbers to each row
  ROW_NUMBER() OVER() AS Row_N
FROM Summer_Medals
ORDER BY Row_N ASC;
```

## Numeração dos jogos olímpicos em ordem crescente
O conjunto de dados dos Jogos Olímpicos de Verão contém os resultados dos jogos entre 1896 e 2012. 
As primeiras Olimpíadas de Verão foram realizadas em 1896, a segunda em 1900 e assim por diante. 
E se você quiser consultar facilmente a tabela para ver em que ano foram realizados os 13º Jogos Olímpicos de Verão? 
Você precisaria numerar as linhas para isso.

```sql
SELECT
  Year,
  -- Assign numbers to each year
  ROW_NUMBER() OVER() AS Row_N
FROM (
  SELECT DISTINCT year
  FROM Summer_Medals
  ORDER BY Year ASC
) AS Years
ORDER BY Year ASC;
```

## Numeração dos jogos olímpicos em ordem decrescente
Você já numerou as linhas no conjunto de dados Summer Medals. 
E se você precisar inverter os números das linhas para que as linhas dos jogos olímpicos mais recentes tenham um número menor?

```sql
SELECT
  Year,
  -- Assign the lowest numbers to the most recent years
   ROW_NUMBER() OVER ( ORDER BY Year DESC) AS Row_N
FROM (
  SELECT DISTINCT Year
  FROM Summer_Medals
) AS Years
ORDER BY Year DESC;
```

## Numeração de atletas olímpicos por medalhas conquistadas
A numeração de linhas também pode ser usada para classificação. 
Por exemplo, ao numerar as linhas e ordenar pela contagem de medalhas que cada atleta ganhou na cláusula OVER, 
você atribuirá 1 ao medalhista que ganhou mais medalhas, 2 ao segundo medalhista que ganhou mais medalhas e assim por diante.

```sql
WITH Athlete_Medals AS (
  SELECT
    -- Count the number of medals each athlete has earned
    Athlete,
    COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Athlete)

SELECT
  -- Number each athlete by how many medals they've earned
  Athlete,
  ROW_NUMBER() OVER (ORDER BY Medals DESC) AS Row_N
FROM Athlete_Medals
ORDER BY Medals DESC;
```

## Os atuais campeões de levantamento de peso
Um campeão em exercício é aquele que venceu as competições do ano anterior e do ano atual. 
Para determinar se um campeão está reinando, os resultados dos anos anterior e atual precisam estar na mesma linha, em duas colunas diferentes.

```sql
WITH Weightlifting_Gold AS (
  SELECT
    -- Return each year's champions' countries
    Year,
    Country AS champion
  FROM Summer_Medals
  WHERE
    Discipline = 'Weightlifting' AND
    Event = '69KG' AND
    Gender = 'Men' AND
    Medal = 'Gold')

SELECT
  Year, Champion,
  -- Fetch the previous year's champion
  lag (Champion) OVER
    (ORDER BY Year ASC) AS Last_Champion
FROM Weightlifting_Gold
ORDER BY Year ASC;

```

## Campeões atuais por gênero
Você já conseguiu o campeão do ano anterior para um evento. No entanto, se você tiver vários eventos, 
gêneros ou outras métricas como colunas, precisará dividir sua tabela em partições para evitar que um campeão de um evento ou gênero apareça como o campeão 
anterior de outro evento ou gênero.
```sql
WITH Tennis_Gold AS (
  SELECT DISTINCT
    Gender, Year, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Event = 'Javelin Throw' AND
    Medal = 'Gold')

SELECT
  Gender, Year,
  Country AS Champion,
  -- Fetch the previous year's champion by gender
  LAG(Country) OVER (PARTITION BY Gender
            ORDER BY Year ASC) AS Last_Champion
FROM Tennis_Gold
ORDER BY Gender ASC, Year ASC;
```

## Campeões atuais por gênero e evento
No exercício anterior, você fez a partição por gênero para garantir que os dados de um gênero não fossem misturados aos dados do outro gênero. 
No entanto, se você tiver várias colunas, o particionamento por apenas uma delas ainda misturará os resultados das outras colunas.
```sql
WITH Athletics_Gold AS (
  SELECT DISTINCT
    Gender, Year, Event, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Discipline = 'Athletics' AND
    Event IN ('100M', '10000M') AND
    Medal = 'Gold')

SELECT
  Gender, Year, Event,
  Country AS Champion,
  -- Fetch the previous year's champion by gender and event
  LAG(Country) OVER (PARTITION BY Gender, Event
            ORDER BY Year ASC) AS Last_Champion
FROM Athletics_Gold
ORDER BY Event ASC, Gender ASC, Year ASC;
```
