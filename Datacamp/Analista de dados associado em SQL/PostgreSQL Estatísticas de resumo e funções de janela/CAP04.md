## Um pivô básico
Você tem a seguinte tabela de países medalhistas de ouro no salto com vara por gênero em 2008 e 2012.

```sql
-- Create the correct extension to enable CROSSTAB
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM CROSSTAB($$
  SELECT
    Gender, Year, Country
  FROM Summer_Medals
  WHERE
    Year IN (2008, 2012)
    AND Medal = 'Gold'
    AND Event = 'Pole Vault'
  ORDER By Gender ASC, Year ASC
-- Fill in the correct column names for the pivoted table
$$) AS ct (Gender VARCHAR,
           "2008" VARCHAR,
           "2012" VARCHAR)

ORDER BY Gender ASC;
```

## Pivotar com classificação
Você quer produzir uma tabela fácil de ser escaneada com as classificações dos três países mais populosos EU de acordo com o número de medalhas de ouro que eles ganharam nos Jogos Olímpicos de 2004 a 2012. 
A tabela precisa estar neste formato:

```sql
-- Count the gold medals per country and year
SELECT
  Country,
  Year,
  COUNT(*) AS Awards
FROM Summer_Medals
WHERE
  Country IN ('FRA', 'GBR', 'GER')
  AND Year IN (2004, 2008, 2012)
  AND Medal = 'Gold'
GROUP BY COUNTRY, Year
ORDER BY Country ASC, Year ASC
```

```sql
WITH Country_Awards AS (
  SELECT
    Country,
    Year,
    COUNT(*) AS Awards
  FROM Summer_Medals
  WHERE
    Country IN ('FRA', 'GBR', 'GER')
    AND Year IN (2004, 2008, 2012)
    AND Medal = 'Gold'
  GROUP BY Country, Year
)

SELECT
  -- Select Country and Year
  Country,
  Year,
  -- Rank by gold medals earned per year
  RANK() OVER (PARTITION BY Year ORDER BY Awards DESC) :: INTEGER AS rank
FROM Country_Awards
ORDER BY Country ASC, Year ASC;


```
## Subtotais em nível de país
Você deseja analisar as medalhas de ouro conquistadas por três países escandinavos, por país e gênero, 
no ano de 2004. Você também está interessado em Country-level subtotals para obter o total de medalhas obtidas por cada país, mas Gender-level subtotals não fazem muito sentido nesse caso, portanto, desconsidere-os.

```sql
-- Count the gold medals per country and gender
SELECT
  Country,
  Gender,
  COUNT(*) AS Gold_Awards
FROM Summer_Medals
WHERE
  Year = 2004
  AND Medal = 'Gold'
  AND Country IN ('DEN', 'NOR', 'SWE')
-- Generate Country-level subtotals
GROUP BY Country, ROLLUP(Gender)
ORDER BY Country ASC, Gender ASC;
```
## Todos os subtotais em nível de grupo
Você deseja dividir todas as medalhas concedidas à Rússia nos Jogos Olímpicos de 2012 por gênero e tipo de medalha. 
Como todas as medalhas pertencem a um país, a Rússia, faz sentido gerar todos os subtotais possíveis (Gender- e Medal-level subtotals), bem como um total geral.

Gerar um detalhamento das medalhas concedidas à Rússia por país e tipo de medalha, 
incluindo todos os subtotais em nível de grupo e um total geral.

```sql
-- Count the medals per gender and medal type
SELECT
  gender,
  Medal,
  count(*) AS Awards
FROM Summer_Medals
WHERE
  Year = 2012
  AND Country = 'RUS'
-- Get all possible group-level subtotals
GROUP BY CUBE(Gender, Medal)
ORDER BY Gender ASC, Medal ASC;
```
## Limpeza dos resultados
Voltando ao detalhamento dos prêmios escandinavos que você fez anteriormente, 
você deseja limpar os resultados substituindo os nulls por um texto significativo.

```sql
SELECT
  -- Replace the nulls in the columns with meaningful text
  COALESCE(Country, 'All countries') AS Country,
  COALESCE(Gender, 'All genders') AS Gender,
  COUNT(*) AS Awards
FROM Summer_Medals
WHERE
  Year = 2004
  AND Medal = 'Gold'
  AND Country IN ('DEN', 'NOR', 'SWE')
GROUP BY ROLLUP(Country, Gender)
ORDER BY Country ASC, Gender ASC;
```
## Resumindo os resultados
Depois de classificar cada país nas Olimpíadas de 2000 por medalhas de ouro concedidas, você deseja retornar os três principais países em uma linha, 
como uma cadeia de caracteres separada por vírgulas. Em outras palavras, vire isso:

```sql
WITH Country_Medals AS (
  SELECT
    Country,
    COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE Year = 2000
    AND Medal = 'Gold'
  GROUP BY Country)

  SELECT
    Country,
    -- Rank countries by the medals awarded
    RANK() OVER (ORDER BY Medals DESC) AS Rank
  FROM Country_Medals
  ORDER BY Rank ASC;
```

```sql
WITH Country_Medals AS (
  SELECT
    Country,
    COUNT(*) AS Medals
  FROM Summer_Medals
  WHERE Year = 2000
    AND Medal = 'Gold'
  GROUP BY Country),

  Country_Ranks AS (
  SELECT
    Country,
    RANK() OVER (ORDER BY Medals DESC) AS Rank
  FROM Country_Medals
  ORDER BY Rank ASC)

-- Compress the countries column
SELECT STRING_AGG(Country, ', ')
FROM Country_Ranks
-- Select only the top three ranks
WHERE Rank <= 3;
```
