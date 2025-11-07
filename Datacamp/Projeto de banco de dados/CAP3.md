## Exibição de views
Como as views são muito úteis, é comum acabar com muitas delas em seu banco de dados. É importante controlá-las para que os usuários do banco de dados saibam o que está disponível para eles.

O objetivo deste exercício é familiarizar-se com a exibição de views em um banco de dados e interpretar sua finalidade.
Essa é uma habilidade necessária para escrever a documentação do banco de dados ou organizar views.

```sql
-- Get all non-systems views
SELECT * FROM information_schema.views
WHERE table_schema not in ('pg_catalog', 'information_schema');

```
## Criação e consulta de uma view
```sql
-- Create a view for reviews with a score above 9
CREATE VIEW high_scores AS
SELECT * FROM reviews
WHERE score > 9;

--
-- Create a view for reviews with a score above 9
CREATE VIEW high_scores AS
SELECT * FROM REVIEWS
WHERE score > 9;

-- Count the number of self-released works in high_scores
SELECT COUNT(*) FROM high_scores
INNER JOIN labels ON high_scores.reviewid = labels.reviewid
WHERE label = 'self-released';

```
## Criação de uma view a partir de outras views
As views podem ser criadas a partir de consultas que incluem outras views. 
Isso é útil quando você tem um esquema complexo, possivelmente devido à normalização, porque ajuda a reduzir os JOINS necessários. 
A maior preocupação é manter o controle das dependências, especificamente como qualquer modificação ou eliminação de uma view pode afetar outras views.
```sql
-- Create a view with the top artists in 2017
CREATE VIEW top_artists_2017 AS
-- with only one column holding the artist field
SELECT artist_title.artist FROM artist_title
INNER JOIN top_15_2017
ON artist_title.reviewid = top_15_2017.reviewid;

-- Output the new view
SELECT * FROM top_artists_2017;
```
## Concessão e revogação de acesso
O controle de acesso é um aspecto fundamental do gerenciamento de bancos de dados. Nem todos os usuários de bancos de dados têm as mesmas necessidades e objetivos, desde analistas, funcionários, cientistas de dados até engenheiros de dados.
Como regra geral, o acesso de gravação nunca deve ser o padrão e só deve ser concedido quando necessário.
```sql
-- Revoke everyone's update and insert privileges
REVOKE UPDATE, INSERT ON long_reviews FROM PUBLIC; 

-- Grant editor update and insert privileges 
GRANT UPDATE, INSERT ON long_reviews TO editor; 
```
## Redefinindo uma view
Ao contrário da inserção e da atualização, redefinir uma view não significa modificar os dados reais que ela contém.
Em vez disso, significa modificar a consulta subjacente que cria a view. 
No último vídeo, aprendemos sobre duas maneiras de redefinir uma view: (1) CREATE OR REPLACE e (2) DROP, então CREATE. CREATE OR REPLACE só pode ser usado sob certas condições.
```sql
-- Redefine the artist_title view to have a label column
CREATE OR REPLACE VIEW artist_title AS
SELECT reviews.reviewid, reviews.title, artists.artist, labels.label
FROM reviews
INNER JOIN artists
ON artists.reviewid = reviews.reviewid
INNER JOIN labels
ON reviews.reviewid = labels.reviewid;

SELECT * FROM artist_title;
```
## Criação e atualização de uma view materializada
A sintaxe para criar views materializadas e não materializadas é bastante semelhante, pois ambas são definidas por uma consulta. 
Uma das principais diferenças é que podemos atualizar views materializadas, enquanto esse conceito não existe para views não materializadas. É importante saber como atualizar uma view materializada, caso contrário, a view continuará sendo um instantâneo do momento em que foi criada.
```sql
-- Create a materialized view called genre_count 
CREATE MATERIALIZED VIEW genre_count AS
SELECT genre, COUNT(*) 
FROM genres
GROUP BY genre;

INSERT INTO genres
VALUES (50000, 'classical');

-- Refresh genre_count
REFRESH MATERIALIZED VIEW genre_count;

SELECT * FROM genre_count;
```
