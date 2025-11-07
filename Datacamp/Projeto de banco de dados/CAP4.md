## Criar uma função
Uma função de banco de dados é uma entidade que contém informações que definem os privilégios da função e interagem com o sistema de autenticação do cliente.
As funções permitem que você conceda níveis diferentes de acesso a pessoas diferentes (e, muitas vezes, a grupos de pessoas) que interagem com seus dados.
```sql
-- Create a data scientist role
CREATE ROLE data_scientist;

--
-- Create a role for Marta
CREATE ROLE marta LOGIN;

--
-- Create an admin role
CREATE ROLE admin WITH CREATEDB CREATEROLE;

```
## GRANT privilégios e atributos ALTER
Depois que as funções são criadas, você concede a elas privilégios específicos de controle de acesso a objetos, como tabelas e views.
Os privilégios comuns são SELECT, INSERT, UPDATE, etc.
```sql
-- Grant data_scientist update and insert privileges
GRANT UPDATE, INSERT ON long_reviews TO data_scientist;

-- Give Marta's role a password
ALTER ROLE marta WITH PASSWORD 's3cur3p@ssw0rd';
```
## Adicionar uma função de usuário a uma função de grupo
Há dois tipos de funções: funções de usuário e funções de grupo. Ao atribuir uma função de usuário a uma função de grupo, 
um administrador de banco de dados pode adicionar níveis complicados de acesso aos seus bancos de dados com um simples comando.
```sql
-- Add Marta to the data scientist group
GRANT data_scientist TO marta;

-- Celebrate! You hired data scientists.

-- Remove Marta from the data scientist group
REVOKE data_scientist FROM marta;
```
## Criação de partições verticais
No vídeo, você aprendeu sobre particionamento vertical e viu um exemplo.

Para o particionamento vertical, não há sintaxe específica no PostgreSQL. Você precisa criar uma nova tabela com colunas específicas e copiar os dados para lá. Depois disso, você pode soltar as colunas que deseja na partição separada. 
Se você precisar acessar a tabela completa, poderá fazê-lo usando uma cláusula JOIN.
```sql
-- Create a new table called film_descriptions
CREATE TABLE film_descriptions (
    film_id INT,
    long_description TEXT
);

-- Copy the descriptions from the film table
INSERT INTO film_descriptions
SELECT film_id, long_description FROM film;
    
-- Drop the column in the original table
ALTER TABLE film DROP COLUMN long_description;

-- Join to create the original table
SELECT * FROM film 
JOIN film_descriptions USING(film_id);
```
## Criação de partições horizontais
No vídeo, você também aprendeu sobre o particionamento horizontal.

O exemplo de particionamento horizontal mostrou a sintaxe necessária para criar partições horizontais no PostgreSQL.
Se precisar de um lembrete, você pode dar uma olhada nos slides.

No entanto, neste exercício, você usará uma partição de lista em vez de uma partição de intervalo. 
Para partições de lista, você forma partições verificando se a chave de partição está em uma lista de valores ou não.
```sql
-- Create a new table called film_partitioned
CREATE TABLE film_partitioned (
  film_id INT,
  title TEXT NOT NULL,
  release_year TEXT
)
PARTITION BY LIST (release_year);

-- Create the partitions for 2019, 2018, and 2017
CREATE TABLE film_2019
	PARTITION OF film_partitioned FOR VALUES IN ('2019');

CREATE TABLE film_2018
	PARTITION OF film_partitioned FOR VALUES IN ('2018');

CREATE TABLE film_2017
	PARTITION OF film_partitioned FOR VALUES IN ('2017');

-- Insert the data into film_partitioned
INSERT INTO film_partitioned
SELECT film_id, title, release_year FROM film;

-- View film_partitioned
SELECT * FROM film_partitioned;
```
