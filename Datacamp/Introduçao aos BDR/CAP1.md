##
```SQL
```
##
```SQL
```
##
```SQL
```
##
```SQL
```
## Consulta no information_schema com SELECT
O information_schema é um metabanco de dados que contém informações sobre o banco de dados atual. O information_schema tem várias tabelas que você pode consultar com a sintaxe conhecida SELECT * FROM:

- tables: informações sobre todas as tabelas do banco de dados atual
- columns: informações sobre todas as colunas de todas as tabelas do banco de dados atual

Neste exercício, você precisa apenas de informações do esquema 'public', que é especificado como a coluna table_schema das tabelas tables e columns. O esquema 'public' contém informações sobre tabelas e bancos de dados definidos pelo usuário. 
Os outros tipos de table_schema contêm informações do sistema.

```SQL
-- Query the right table in information_schema
SELECT table_name 
FROM information_schema.tables
-- Specify the correct table_schema value
WHERE table_schema = 'public';
```

## Consulta no `information_schema` com `SELECT`

O `information_schema` é um metabanco de dados que contém informações sobre o banco de dados atual. Ele possui diversas tabelas internas que podem ser consultadas usando a sintaxe padrão:

SELECT * FROM information_schema.<tabela>;

- **tables**: contém informações sobre todas as tabelas do banco de dados atual.  
- **columns**: contém informações sobre todas as colunas de todas as tabelas do banco de dados.

```SQL
-- Query the right table in information_schema to get columns
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'university_professors' AND table_schema = 'public';
```
## CREATE Você está fazendo as primeiras TABELAS
Agora você vai começar a implementar um modelo de banco de dados melhor. 
Para isso, deve criar tabelas para os tipos de entidade professors e universities. As outras tabelas serão criadas para você.

```SQL
-- Create a table for the professors entity type
CREATE TABLE professors (
 firstname text,
 lastname text
);

-- Print the contents of this table
SELECT * 
FROM professors
```
## Adicione uma coluna com ALTER TABLE
```SQL
-- Add the university_shortname column
ALTER TABLE professors
ADD COLUMN university_shortname text;

-- Print the contents of this table
SELECT * 
FROM professors
```

## RENAME e DROP COLUMNs em afiliações
```SQL
-- Rename the organisation column
ALTER TABLE affiliations
RENAME COLUMN organisation TO organization;

-- Delete the university_shortname column
ALTER TABLE affiliations
DROP COLUMN university_shortname;
```

## Migre dados com INSERT INTO SELECT DISTINCT
```SQL
-- Insert unique professors into the new table
INSERT INTO professors 
SELECT DISTINCT firstname, lastname, university_shortname 
FROM university_professors;

-- Doublecheck the contents of professors
SELECT * 
FROM professors;

------

-- Insert unique affiliations into the new table
INSERT INTO affiliations 
SELECT DISTINCT firstname, lastname, function, organization 
FROM university_professors;

-- Doublecheck the contents of affiliations
SELECT * 
FROM affiliations;
```
## Exclua tabelas com DROP TABLE
```SQL
-- Delete the university_professors table
DROP TABLE university_professors;
```
