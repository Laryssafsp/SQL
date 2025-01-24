# [DIO - SQL Tuning](https://www.youtube.com/watch?v=s20CgcM8gtk)

- Entender os fundamentos da otimização de consultas SQL
- Identificar gargalos de desempenho em consultas
- Aplicação de técnicas práticas para otimização de consultas
- Uso de ferramentas práticas como EXPLAIN para análise

###  Introdução SQL Tunning
SQL Tuning é o processo de ajuste de consultas SQL para melhorar o desempenho
no banco de dados.

Performance e Escalabilidade:
- Consultas mal otimizadas podem ser muito lentas, especialmente em sistemas que lidam com grandes volumes de dados.
- Uma consulta ineficiente pode consumir (muitos) recursos desnecessários.

Impacto nos Negócios:
- A lentidão em sistemas pode afetar a experiência do usuário (e afetar as receitas).
- Empresas podem ter em custos elevados de infraestrutura para compensar consultas ineficiente, e não só isso, mas perda de consumidores e clientes.

|CODE SQL                  |TIME                        |
|----------------------------------------------:|:-------------------------------:|
|SELECT * FROM results_copa | 5.41s runtime|
|SHOW COLUMNS FROM results_copa| 241 rows / 0.29s runtime|
|SELECT numero documento de venda, currency type FROM results copa | 1.03s runtime|

Tivemos uma melhoria significativa (425% !! ) no tempo de processamento e utilização de capacidade, mas podemos otimizar ainda mais!


### Fundamentos da otimização

- Plano de Execução (EXPLAIN/EXPLAIN FORMATTED)
- Demonstra como o banco de dados executa uma consulta SQL, detalhando as etapas e os recursos utilizados.

#### Explain
  `EXPLAIN FORMATTED SELECT numero_documento_de_venda, currency_type FROM results_copa WHERE numero_documento_de_venda = 12345`

#### Uso de índices: B-Tree, Full-Text, Hash index [hierarquia balanceada]  - retira a necessidade implícita de ordenar a tabela
- B-Tree: Excelente para buscas exatas ou intervalares.
- Full-Text: Ideal para buscas em textos longos.
- Hash Indexes: Focados em valores exatos, sem suporte para intervalos.

##### Índice B-Tree :: funciona melhor em colunas númericas com ranges e calculos matemáricos

```
-- Criando o índice B-Tree na coluna 'numero_documento_de_venda'
CREATE INDEX idx_numero_documento_de_venda
ON results_copa (numero_documento_de_venda);
```
Índice B-Tree - O índice B-Tree reduz a necessidade de escanear a tabela inteira, localizando o valor `12345 diretamente.
```
SELECT numero_documento_de_venda, currency_type
FROM results_copa
WHERE numero_documento_de_venda = 12345;
```

O índice permite localizar rapidamente os documentos no intervalo definido e evita a necessidade de ordenação explícita, já que o B-Tree mantém os valores em ordem.
```
-- Busca de documentos dentro de um intervalo
SELECT numero_documento_de_venda, currency_type
FROM results_copa
WHERE numero_documento_de venda BETWEEN 10000 AND 20000
```

##### Indice Full-Text
```
Criando o indice Full-Text na coluna `material'
CREATE fullltext INDEX idx_material
ON results_copa (material);
```

O Full-Text Index permite buscas rápidas de palavras específicas em textos grandes, identificando os documentos que mencionam "Heineken".
```
SELECT numero_documento_de_venda, material
FROM results_copa
WHERE MATCH(material) AGAINST ('Heineken' IN NATURAL LANGUAGE MODE);
```

Busca documentos que contenham ambas as palavras.
```
-- Consultando documentos que contenham "Heineken" e "Retornavel"
SELECT numero_documento_de_venda, material
FROM results_copa
WHERE MATCH(material) AGAINST ('+Heineken +Retornavel' IN BOOLEAN MODE);
```

|Modo                |Quando Usar                        |
|----------------------------------------------:|:-------------------------------:|
BOOLEAN MODE| Quando precisa de controle preciso sobre a busca e resultados.
NATURAL LANGUAGE MODE| Quando a prioridade é a relevância geral, sem exigir correspondências exatas.|


##### Índice Hash
```
-- Criando oindice Hash
CREATE INDEX idx_hash_numero_documento_de_venda
USING HASH
ON results_copa (numero_documento_de_venda);
```

O índice Hash encontra rapidamente a chave exata.

```
-- Consulta aproveitando o indice Hash
SELECT numero_documento de venda, material
FROM results_copa
WHERE numero documento de_venda = 12345;
```
OBS.: NÃO funciona para intervalos (BETWEEN) ou ordenação.

#### Resumo aplicações

|Tipo de Índice  |Quando Usar                        | Caso de Uso
|----------------------------------------------:|:-------------------------------:|:-------------------------------:|
B-Tree    | Busca exata, intervalar ou ordenação | WHERE numero_documento_de_venda BETWEEN 100 AND 200
Full-Text | Busca em textos longos               | MATCH(material) AGAINST ('palavra-chave')
Hash      | Busca exata (valores únicos)         | WHERE numero_documento_de_venda = 12345

####  Evitar operações que invalidem índices (funções em colunas) - Exemplo: UPPER(coluna)

```
SELECT numero_documento_de_venda
FROM results_copa
WHERE UPPER (material,'HEINEKEN';
```

#### O banco de dados precisa calcular UPPER(material) para cada linha antes de comparar, resultando em um table scan.

- Especificidade: Use colunas específicas ao invés de SELECT *
- Carrega colunas desnecessárias, aumentando o uso de memória
- Pode impactar performance ao longo do tempo com mudanças no schema.

### Técnicas Práticas

#### Filtrar dados de forma eficiente
- Combinar filtros com índices e utilizar condições otimizadas (WHERE, BETWEEN, IN, etc.)

#### Subconsultas x JOINs: qual usar?
- Subconsultas : Úteis quando você precisa de uma relação isolada ou calcular um onjunto de dados para ser usado no filtro principal (use com cautela)
- Join: Ideal para unir tabelas relacionadas e evitar avaliações repetidas. (sempre mais performático)

SUBQUERY
```
-- USE SEMPRE JOINs AO INVÉS DE SUBQUERIES: ~30% MAIS EFICIÊNCIA COM MÍNIMAS MODIFICAÇÕES

SELECT nome, (SELECT SUM(valor_total)
FROM pedidos
WHERE pedidos.cliente_id = clientes.id) AS total_gasto
FROM clientes;
```
JOIN
```
SELECT clientes.nome, SUM(pedidos.valor_total) AS total gasto
FROM clientes
JOIN pedidos ON clientes.id = pedidos.cliente_id
GROUP BY clientes.nome;
```

#### Criar índices em colunas filtradas e JOINs
- Colunas usadas em condições de filtro (WHERE) e em JOINs se beneficiam muito de ndices, tornando as operações mais rápidas
- Excesso de índices causam lentidão no INSERT? Sim! Analisar Caso o nº de inserts seja menor que a consulta de registro por dias, analisar o custo/benefício pois é o fator determinante para  a estratégia de otimização. Bancos mais transacionais, ou seja, que tem mais insets do que consultas terão menos índices. Quando é uma base análitica, insert menor do que select, a necessidade de criação de índices é maior.
```
-- CRIE ÍNDICES EM COLUNAS QUE SERÃO FREQUENTEMENTE USADAS EM FILTROS
SELECT p.id, p.valor_total, c.nome
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
WHERE c.cidade = 'Sao Paulo';
```
= Tempo de resposta: 552.5s

Criando indíce e realizando a mesma consulta:
```
CREATE INDEX idx_cliente_id ON pedidos(cliente_id);
CREATE INDEX idx_cliente_cidade ON clientes(cidade);
```
= Tempo de resposta: 3.2s (-16k%)

#### Resumo e Próximos Passos
- Interpretar planos de execução
  - Saber interpretar um plano de execução ajuda a identificar gargalos e otimizar consultas

- Aproveitar índices corretamente
  - Índices permitem buscas rápidas, semelhantes a um índice de um livro

- Evitar práticas que invalidem índices
  - Mesmo com índices criados, certas práticas podem invalidá-los, fazendo com que o banco execute uma varredura completa da tabela

-  Materiais recomendados:
  - [Use the Index, Luke!](https://use-the-index-luke.com/)
  - [Documentacao oficial EXPLAIN (PostgreSQL/MySQL)](https://www.postgresql.org/docs/current/sql-explain.html)




