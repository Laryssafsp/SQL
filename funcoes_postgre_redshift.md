📣 Sugestão de publicação para o LinkedIn:
👨‍💻 Está começando com SQL em PostgreSQL ou Redshift?
Preparei um mini manual em formato de tabela com funções básicas, intermediárias e avançadas — incluindo aquelas pouco exploradas, mas poderosas 💥

Se você já domina SELECT e GROUP BY, que tal explorar funções como LAG(), GENERATE_SERIES() ou PERCENTILE_CONT() para levar suas análises a outro nível?

📊 Tabelinha completa abaixo 👇
Salva e compartilha com quem também quer turbinar o SQL!
#SQL #PostgreSQL #Redshift #DataAnalytics #BusinessIntelligence #SQLTips


📘 TABELA ATUALIZADA – FUNÇÕES SQL QUE TODO ANALISTA DEVERIA CONHECER

🚀 Essenciais e muito usadas

| Função/Comando    | Descrição rápida                  | Exemplo                                     |
| ----------------- | --------------------------------- | ------------------------------------------- |
| `SELECT`          | Seleciona dados                   | `SELECT * FROM tabela;`                     |
| `WHERE`           | Filtra linhas                     | `WHERE status = 'ativo'`                    |
| `GROUP BY`        | Agrupa resultados para agregações | `GROUP BY cliente_id`                       |
| `COUNT()`         | Conta registros                   | `COUNT(*)`                                  |
| `SUM()` / `AVG()` | Soma / Média de valores numéricos | `SUM(valor)`                                |
| `JOIN`            | Une tabelas                       | `INNER JOIN`, `LEFT JOIN`                   |
| `CASE WHEN`       | Condicional (tipo IF)             | `CASE WHEN x > 0 THEN 'ok' ELSE 'erro' END` |
| `ORDER BY`        | Ordena o resultado                | `ORDER BY data DESC`                        |
| `DISTINCT`        | Remove duplicados                 | `SELECT DISTINCT nome`                      |
| `LIMIT`           | Limita nº de linhas retornadas    | `LIMIT 100`                                 |


| Função                           | Descrição prática                                                   | Exemplo                                                                    |
| -------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `COALESCE()`                     | Retorna o **primeiro valor não nulo** entre os argumentos           | `COALESCE(coluna, 'sem valor')`                                            |
| `NULLIF()`                       | Retorna NULL se os dois argumentos forem iguais                     | `NULLIF(valor1, valor2)`                                                   |
| `LEAD()` / `LAG()`               | Acessa o valor da próxima ou anterior linha (útil em janelas)       | `LAG(saldo) OVER (PARTITION BY cliente ORDER BY data)`                     |
| `RANK()` / `DENSE_RANK()`        | Numera linhas por partição (ranking com empates ou não)             | `RANK() OVER (PARTITION BY produto ORDER BY vendas DESC)`                  |
| `GENERATE_SERIES()`              | Gera intervalos de datas/números — muito útil para séries temporais | `SELECT * FROM generate_series('2023-01-01'::date, '2023-01-10', '1 day')` |
| `DATE_TRUNC()`                   | Trunca uma data para mês, dia, hora...                              | `DATE_TRUNC('month', data)`                                                |
| `EXTRACT()`                      | Extrai partes de data (ano, mês, etc.)                              | `EXTRACT(MONTH FROM data)`                                                 |
| `TO_CHAR()`                      | Formata data ou número como texto                                   | `TO_CHAR(data, 'YYYY-MM-DD')`                                              |
| `PERCENTILE_CONT()`              | Calcula percentis (ex: mediana)                                     | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY valor)`                       |


🧩 Manipulação de JSON

| Função                     | Para quê serve          | Exemplo                                     |
| -------------------------- | ----------------------- | ------------------------------------------- |
| `->`, `->>`                | Acessa campos JSON      | `json_col ->> 'nome'`                       |
| `#>>`                      | Acessa campos aninhados | `json_col #>> '{endereco, cidade}'`         |
| `json_extract_path_text()` | Extrai campo (Redshift) | `json_extract_path_text(json_col, 'id')`    |
| `jsonb_array_elements()`   | Explode arrays JSON     | `jsonb_array_elements(json_col -> 'itens')` |

🔎 Textos, expressões e outros coringas

| Função                           | Para quê serve                    | Exemplo                                   |
| -------------------------------- | --------------------------------- | ----------------------------------------- |
| `REGEXP_REPLACE()`               | Substituição usando regex         | `REGEXP_REPLACE(nome, '[^a-z]', '', 'g')` |
| `ILIKE`                          | Comparação sem case sensitive     | `WHERE nome ILIKE '%joão%'`               |
| `TRIM()` / `LTRIM()` / `RTRIM()` | Remove espaços extras             | `TRIM(coluna)`                            |
| `TO_CHAR()`                      | Formata data ou número como texto | `TO_CHAR(data, 'YYYY-MM')`                |
| `CAST()` / `::`                  | Converte tipo de dado             | `CAST(valor AS TEXT)` ou `valor::text`    |
