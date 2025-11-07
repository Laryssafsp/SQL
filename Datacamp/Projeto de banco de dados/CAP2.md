## Seus aprendizados recentes
Quando você saiu há 12 horas, você trabalhou em Database Schemas and Normalization, capítulo 2 do curso Projeto de banco de dados. Veja o que você abordou na última lição:

Você aprendeu sobre técnicas avançadas de modelagem de dados, focando nos esquemas estrela e floco de neve, e no conceito de normalização em bancos de dados. Aqui está um resumo rápido:

- **Esquema Estrela**: Você descobriu que o esquema estrela é um tipo de modelo dimensional que inclui uma tabela de fatos central cercada por tabelas de dimensão. A tabela de fatos contém dados quantitativos (como valor de vendas e quantidade), enquanto as tabelas de dimensão descrevem atributos dos dados (como detalhes do livro, tempo de venda e informações da loja). Este esquema é nomeado por sua forma de estrela, onde cada tabela de dimensão se liga diretamente à tabela de fatos.

- **Esquema Floco de Neve**: Você aprendeu que o esquema floco de neve é uma extensão do esquema estrela, onde as tabelas de dimensão são normalizadas em várias tabelas relacionadas. Isso reduz a redundância e melhora a integridade dos dados, dividindo os dados em partes menores e mais gerenciáveis. O esquema floco de neve é mais complexo devido às suas camadas adicionais de tabelas de dimensão.

- **Normalização**: O processo de normalização envolve dividir tabelas em tabelas menores e relacionadas para reduzir a redundância de dados e aumentar a integridade dos dados. Você viu exemplos de como grupos repetidos de dados, como autores e gêneros em uma dimensão de livro, podem ser separados em suas próprias tabelas em um esquema floco de neve.

- **Aplicação Prática**: Através de exercícios, você aplicou esses conceitos transformando um esquema estrela em um esquema floco de neve para um banco de dados em execução. Você também praticou adicionar chaves estrangeiras para conectar tabelas de dimensão a uma tabela de fatos, reforçando a importância dessas chaves na manutenção das relações entre as tabelas.

Aqui está um trecho de um dos exercícios onde você adicionou chaves estrangeiras à tabela fact_booksales:

```sql
-- Adicionar a chave estrangeira book_id
ALTER TABLE fact_booksales ADD CONSTRAINT sales_book
    FOREIGN KEY (book_id) REFERENCES dim_book_star (book_id);
    
-- Adicionar a chave estrangeira time_id
ALTER TABLE fact_booksales ADD CONSTRAINT sales_time
   FOREIGN KEY (time_id) REFERENCES dim_time_star (time_id);
    
-- Adicionar a chave estrangeira store_id
ALTER TABLE fact_booksales ADD CONSTRAINT sales_store
   FOREIGN KEY (store_id) REFERENCES dim_store_star (store_id);
```
Esta lição equipou você com o conhecimento para projetar bancos de dados mais eficientes e organizados usando esquemas estrela e floco de neve, juntamente com uma compreensão fundamental das técnicas de normalização.

## Adição de chaves estrangeiras
As referências de chave estrangeira são essenciais para os esquemas em floco de neve e em estrela.
Ao criar qualquer um desses esquemas, é fundamental configurar corretamente as chaves estrangeiras, pois elas conectam as dimensões à tabela de fatos. 
Elas também impõem um relacionamento de um para muitos, pois, a menos que especificado de outra forma, uma chave estrangeira pode aparecer mais de uma vez em uma tabela e a chave primária pode aparecer apenas uma vez
```sql
-- Add the book_id foreign key
ALTER TABLE fact_booksales ADD CONSTRAINT sales_book
    FOREIGN KEY (book_id) REFERENCES dim_book_star (book_id);
    
-- Add the time_id foreign key
ALTER TABLE fact_booksales ADD CONSTRAINT sales_time
   FOREIGN KEY (time_id) REFERENCES dim_time_star (time_id);
    
-- Add the store_id foreign key
ALTER TABLE fact_booksales ADD CONSTRAINT sales_store
   FOREIGN KEY (store_id) REFERENCES dim_store_star (store_id);
```
## Extensão da dimensão do livro

```sql
-- Create a new table for dim_author with an author column
CREATE TABLE dim_author (
    author varchar(256)  NOT NULL
);

-- Insert authors 
INSERT INTO dim_author
SELECT DISTINCT author FROM dim_book_star;

-- Add a primary key 
ALTER TABLE dim_author ADD COLUMN author_id SERIAL PRIMARY KEY;

-- Output the new table
SELECT * FROM dim_author;
```

## Bancos de dados normalizados e desnormalizados
Bancos de dados normalizados economizam espaços, porém mantem muitas repetições.

## Consulta ao esquema em estrela
```sql
-- Output each state and their total sales_amount
SELECT dim_store_star.state, SUM(sales_amount)
FROM fact_booksales
	-- Join to get book information
    JOIN dim_book_star ON fact_booksales.book_id = dim_book_star.book_id
	-- Join to get store information
    JOIN dim_store_star ON fact_booksales.store_id = dim_store_star.store_id
-- Get all books with in the novel genre
WHERE  
    dim_book_star.genre = 'novel'
-- Group results by state
GROUP BY
    dim_store_star.state;
```
## Consulta ao esquema em floco de neve

```sql
-- Output each state and their total sales_amount
SELECT dim_state_sf.state, sum(sales_amount)
FROM fact_booksales
    -- Joins for the genre
    JOIN dim_book_sf on fact_booksales.book_id = dim_book_sf.book_id
    JOIN dim_genre_sf on dim_book_sf.genre_id = dim_genre_sf.genre_id
    -- Joins for the state 
    JOIN dim_store_sf on fact_booksales.store_id = dim_store_sf.store_id 
    JOIN dim_city_sf on dim_store_sf.city_id = dim_city_sf.city_id
	JOIN dim_state_sf on  dim_city_sf.state_id = dim_state_sf.state_id
-- Get all books with in the novel genre and group the results by state
WHERE  
    dim_genre_sf.genre = 'novel'
GROUP BY
   dim_state_sf.state;
```
## Atualização de países 
```sql
-- Output records that need to be updated in the star schema
SELECT * FROM dim_store_star
WHERE country != 'USA' AND country !='CA';
```
## Extensão do esquema em floco de neve
```sql
-- Add a continent_id column with default value of 1
ALTER TABLE dim_country_sf
ADD continent_id int NOT NULL DEFAULT(1);

-- Add the foreign key constraint
ALTER TABLE dim_country_sf 
ADD CONSTRAINT country_continent
FOREIGN KEY (continent_id) REFERENCES dim_continent_sf(continent_id);
   
-- Output updated table
SELECT * FROM dim_country_sf;
```
## Conversão para 1NF
Tabela om ário valores repetidos
```sql
-- Create a new table to hold the cars rented by customers
CREATE TABLE cust_rentals (
  customer_id  INT NOT NULL,
  car_id VARCHAR(128) NULL,
  invoice_id VARCHAR(128) NULL
);

-- Drop two columns from customers table to satisfy 1NF
ALTER TABLE customers
DROP COLUMN customer_name,
DROP COLUMN cars_rented;

```
## Conversão para 2NF
Crie uma nova tabela para as colunas não-chave que estavam em conflito com 2 critériosNF.

Aí está! model, manufacturer, type_car, conditions e colors dependem de car_id, mas são independentes das outras duas chaves primárias, customer_id e start_date. O cliente ou a data de início não podem alterar esses atributos.
Portanto, colocamos essas colunas em uma nova tabela e as removemos de customer_rentals.
```sql
-- Create a new table to satisfy 2NF
Create TABLE cars (
  car_id VARCHAR(256) NULL,
  model VARCHAR(128),
  manufacturer VARCHAR(128),
  type_car VARCHAR(128),
  condition VARCHAR(128),
  color VARCHAR(128)
);

-- Insert data into the new table
INSERT INTO cars
SELECT DISTINCT
  car_id,
  model,
  manufacturer,
  type_car,
  condition,
  color
FROM customer_rentals;

-- Drop columns in customer_rentals to satisfy 2NF
ALTER TABLE customer_rentals
DROP COLUMN model,
DROP COLUMN manufacturer, 
DROP COLUMN type_car,
DROP COLUMN condition,
DROP COLUMN color;
```
## Conversão para 3NF
Crie uma nova tabela para as colunas não-chave que estavam em conflito com 3 critériosNF.

```sql
-- Create a new table to satisfy 3NF
CREATE TABLE car_model(
  model VARCHAR(128),
  manufacturer VARCHAR(128),
  type_car VARCHAR(128)
);

-- Drop columns in rental_cars to satisfy 3NF
ALTER TABLE rental_cars
DROP COLUMN manufacturer, 
DROP COLUMN type_car;
```
