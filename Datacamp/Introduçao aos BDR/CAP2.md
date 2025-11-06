## Conformidade com tipos de dados
```SQL
-- Let's add a record to the table
INSERT INTO transactions (transaction_date, amount, fee) 
VALUES ('2018-09-24', 5454, '30');

-- Doublecheck the contents
SELECT *
FROM transactions;
```
## Tipo CASTS
```SQL
-- Calculate the net amount as amount + fee
SELECT transaction_date, amount + CAST(fee AS INT) AS net_amount 
FROM transactions;
```
## Altere tipos com ALTER COLUMN
```SQL
-- Select the university_shortname column
SELECT DISTINCT(university_shortname) 
FROM professors;

-- Specify the correct fixed-length character type
ALTER TABLE professors
ALTER COLUMN university_shortname
TYPE char(3);

-- Change the type of firstname
ALTER TABLE professors
ALTER COLUMN firstname
TYPE VARchar(64);
```
## Converta tipos usando uma função
Se não quiser reservar muito espaço para uma determinada coluna varchar, você pode truncar os valores antes de converter o tipo
`sql
ALTER TABLE table_name
ALTER COLUMN column_name
TYPE varchar(x)
USING SUBSTRING(column_name FROM 1 FOR x)
`
A forma de ler é assim: como você deseja reservar apenas x caracteres para column_name, é necessário manter uma SUBSTRING de cada valor,
ou seja, os primeiros x caracteres dela, e descartar o restante. Dessa forma, os valores se ajustarão aos requisitos devarchar(x).
```SQL
-- Convert the values in firstname to a max. of 16 characters
ALTER TABLE professors 
ALTER COLUMN firstname 
TYPE varchar(16) 
USING SUBSTRING(firstname FROM 1 FOR 16);

```
## Como evitar valores nulos com SET NOT NULL
```SQL
-- Disallow NULL values in firstname
ALTER TABLE professors 
ALTER COLUMN lastname SET NOT NULL;
```
## Torne suas colunas exclusivas com ADD CONSTRAINT
Se você quiser adicionar uma restrição exclusiva a uma tabela existente, faça isso desta forma:
`
ALTER TABLE table_name
ADD CONSTRAINT some_name UNIQUE(column_name);
`
```SQL
-- Make universities.university_shortname unique
ALTER TABLE  universities
ADD CONSTRAINT university_shortname_unq UNIQUE(university_shortname);
```
