## Introdução a SELECT COUNT DISTINCT
```SQL
-- Count the number of distinct values in the university_city column
SELECT COUNT(DISTINCT(university_city)) 
FROM universities;
```
## Identifique chaves com SELECT COUNT DISTINCT
Há uma maneira muito básica de descobrir o que se qualifica como chave em uma tabela já existente e preenchida:

- Conte os registros distintos de todas as combinações possíveis de colunas. Se o número resultante x for igual ao número de todas as linhas da tabela referente a uma combinação, você descobriu uma superchave.
-  Em seguida, remova uma coluna após a outra até que você não consiga mais remover colunas sem ver o número x diminuir. Se esse for o caso, você descobriu uma chave (candidata).

```SQL
-- Try out different combinations
SELECT COUNT(distinct(firstname, lastname)) 
FROM professors; 
```
## ADD CONSTRAINTs de chave para as tabelas
`
ALTER TABLE table_name
ADD CONSTRAINT some_name PRIMARY KEY (column_name)
`
```SQL
-- Rename the organization column to id
ALTER TABLE organizations
Rename column organization TO id;

-- Make id a primary key
ALTER TABLE organizations
ADD CONSTRAINT organization_pk PRIMARY  KEY (id);
```
## Adicione uma chave alternativa SERIAL

```SQL
-- Add the new column to the table
ALTER TABLE professors 
ADD COLUMN id serial;

-- Make id a primary key
ALTER TABLE professors 
ADD CONSTRAINT professors_pkey PRIMARY KEY (id);

-- Have a look at the first 10 rows of professors
SELECT * FROM professors
LIMIT 10;
```
## CONCATENAR colunas em uma chave substituta
Outra estratégia para adicionar uma chave alternativa a uma tabela existente é concatenar as colunas existentes com a função CONCAT().
`
CREATE TABLE cars (
 make varchar(64) NOT NULL,
 model varchar(64) NOT NULL,
 mpg integer NOT NULL
)
`
```SQL
-- Count the number of distinct rows with columns make, model
SELECT COUNT(DISTINCT(make, model)) 
FROM cars;

-- Add the id column
ALTER TABLE cars
ADD COLUMN id varchar(128);

-- Update id with make + model
UPDATE cars
SET id = CONCAT(make, model);

-- Make id a primary key
ALTER TABLE cars
ADD CONSTRAINT id_pk PRIMARY KEY(ID);

-- Have a look at the table
SELECT * FROM cars;

--

-- Create the table
CREATE TABLE students (
  last_name varchar(128) NOT NULL,
  ssn INTEGER PRIMARY KEY, -- (social security number
  phone_no CHAR(12)
);
```
