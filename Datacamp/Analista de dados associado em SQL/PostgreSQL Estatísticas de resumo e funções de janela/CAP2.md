## Futuros medalhistas de ouro
As funções de busca permitem que você obtenha valores de diferentes partes da tabela em uma linha. 
Se você tiver dados ordenados por tempo, poderá "olhar para o futuro" com a função de busca LEAD. 
Isso é especialmente útil se você quiser comparar um valor atual com um valor futuro.
```sql
WITH Discus_Medalists AS (
  SELECT DISTINCT
    Year,
    Athlete
  FROM Summer_Medals
  WHERE Medal = 'Gold'
    AND Event = 'Discus Throw'
    AND Gender = 'Women'
    AND Year >= 2000)

SELECT
  -- For each year, fetch the current and future medalists
  Year,
  Athlete,
  lead(Athlete,3) OVER (ORDER BY Athlete ASC) AS Future_Champion
FROM Discus_Medalists
ORDER BY Year ASC;
```

## Nome do primeiro atleta
Geralmente, é útil obter o primeiro ou o último valor em um conjunto de dados para comparar todos os outros valores com ele. 
Com funções de busca absoluta como FIRST_VALUE, você pode buscar um valor em uma posição absoluta na tabela, como no início ou no fim.
```sql
WITH All_Male_Medalists AS (
  SELECT DISTINCT
    Athlete
  FROM Summer_Medals
  WHERE Medal = 'Gold'
    AND Gender = 'Men')

SELECT
  -- Fetch all athletes and the first athlete alphabetically
  Athlete,
  FIRST_VALUE(Athlete) OVER (
    ORDER BY Athlete ASC
  ) AS First_Athlete
FROM All_Male_Medalists;

```

## Último país por nome
Assim como é possível obter o valor da primeira linha em um conjunto de dados, ocê pode obter o valor da última linha.
Isso geralmente é útil quando você deseja comparar o valor mais recente com os valores anteriores.

```sql
WITH Hosts AS (
  SELECT DISTINCT Year, City
    FROM Summer_Medals)

SELECT
  Year,
  City,
  -- Get the last city in which the Olympic games were held
  LAST_VALUE (City) OVER (
   ORDER BY City ASC
   RANGE BETWEEN
     UNBOUNDED PRECEDING AND
     UNBOUNDED FOLLOWING
  ) AS Last_City
FROM Hosts
ORDER BY Year ASC;
```

## Classificação dos atletas por medalhas conquistadas
No capítulo 1, você usou o site ROW_NUMBER para classificar os atletas de acordo com as medalhas conquistadas. No entanto, o site ROW_NUMBER atribui números diferentes a atletas com o mesmo número de medalhas concedidas, portanto, não é uma função de classificação útil; se dois atletas ganharam o mesmo número de medalhas, eles devem ter a mesma classificação.

```sql
WITH Athlete_Medals AS (
  SELECT
    Athlete,
    COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Athlete)

SELECT
  Athlete,
  Medals,
  -- Rank athletes by the medals they've won
  Rank() OVER (ORDER BY Medals DESC) AS Rank_N
FROM Athlete_Medals
ORDER BY Medals DESC;
```
## Classificação de atletas de vários países
No exercício anterior, você usou o site RANK para atribuir classificações a um grupo de atletas. No entanto, nos dados do mundo real, você frequentemente encontrará vários grupos dentro dos dados. Sem particionar seus dados, os valores de um grupo influenciarão as classificações dos outros.

Além disso, embora o site RANK ignore números no caso de valores idênticos, a maneira mais natural de atribuir classificações é não ignorar números. Se dois países estiverem empatados em segundo lugar, o país depois deles é considerado o terceiro pela maioria das pessoas.

```sql
WITH Athlete_Medals AS (
  SELECT
    Country, Athlete, COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE
    Country IN ('JPN', 'KOR')
    AND Year >= 2000
  GROUP BY Country, Athlete
  HAVING COUNT(*) > 1)

SELECT
  Country,
  -- Rank athletes in each country by the medals they've won
  Athlete,
  DENSE_RANK() OVER (PARTITION BY Country
    ORDER BY Medals DESC) AS Rank_N
FROM Athlete_Medals
ORDER BY Country ASC, RANK_N ASC;
```
## Eventos de paginação
Há exatamente 666 eventos exclusivos no conjunto de dados das Medalhas Olímpicas de Verão. Se você quiser dividi-los para analisá-los peça por peça, precisará dividir os eventos em grupos de tamanho aproximadamente igual.

```sql
WITH Events AS (
  SELECT DISTINCT Event
  FROM Summer_Medals)
  
SELECT
  --- Split up the distinct events into 111 unique groups
  Event,
  NTILE(111) OVER (ORDER BY Event ASC) AS Page
FROM Events
ORDER BY Event ASC;
```
## Terços superior, médio e inferior
Dividir seus dados em terços ou quartis geralmente é útil para que você entenda como os valores do conjunto de dados estão distribuídos. Obter estatísticas resumidas (médias, somas, desvios padrão, etc.) dos terços superior, médio e inferior pode ajudar você a determinar a distribuição que seus valores seguem.

```sql
WITH Athlete_Medals AS (
  SELECT Athlete, COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Athlete
  HAVING COUNT(*) > 1)
  
SELECT
  Athlete,
  Medals,
  -- Split athletes into thirds by their earned medals
  NTILE(3) OVER (ORDER BY Medals ASC) AS Third
FROM Athlete_Medals
ORDER BY Medals DESC, Athlete ASC;
```
```WITH Athlete_Medals AS (
  SELECT Athlete, COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Athlete
  HAVING COUNT(*) > 1),
  
  Thirds AS (
  SELECT
    Athlete,
    Medals,
    NTILE(3) OVER (ORDER BY Medals DESC) AS Third
  FROM Athlete_Medals)
  
SELECT
  -- Get the average medals earned in each third
  third,
  AVG(Medals) AS Avg_Medals
FROM Thirds
GROUP BY Third
ORDER BY Third ASC;sql
```
