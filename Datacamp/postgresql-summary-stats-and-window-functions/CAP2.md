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

## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
## 

```sql

```
