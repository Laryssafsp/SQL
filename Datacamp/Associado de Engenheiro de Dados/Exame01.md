# Documentação Consolidada: SQL, Banco de Dados e Visualização de Dados

## 1. Modelagem de Dados

* **Entidade `Car` no modelo conceitual**: Representa o **conceito do mundo real de um carro**, abstraindo informações como marca, modelo, ano e cor.

* **Transição do modelo lógico para o físico**: Foco em **otimizar desempenho**, definindo índices, tipos de dados e estratégias de armazenamento.

## 2. Serviços de Nuvem e Pipelines de Dados

* **Google Cloud Composer**: Criação, programação e monitoramento de workflows e pipelines.
* **Azure Data Factory**: Integração de dados (ETL/ELT).
* **Google BigQuery**: Armazenamento colunar para análise rápida.
* **Amazon Kinesis**: Pipeline de dados em tempo real.

## 3. Consultas SQL e Operações com Dados

### 3.1 Conversão de Tipos

```sql
SELECT TO_DATE(delivery_date, 'YYYYMMDD') AS data
FROM sales;
```

### 3.2 Expressões Regulares

```sql
SELECT username, first_name, last_name
FROM users
WHERE username ~ '[0-9]$';

UPDATE accounts
SET username = REGEXP_REPLACE(username, '[0-9]', '', 'g');
```

### 3.3 Agregações e Subqueries

```sql
SELECT custID, creditLine, promoCode
FROM accounts
WHERE promoCode ~ '^[1-3].*[5-9]$';

SELECT branch_id, type, AVG(amount)::DECIMAL(6,2) AS mean_amount
FROM withdrawals
GROUP BY branch_id, type
ORDER BY type, branch_id;

SELECT profile_id, month, SUM(visit_count) AS global_total
FROM (
    SELECT * FROM visits_AMER
    UNION ALL
    SELECT * FROM visits_EMEA
    UNION ALL
    SELECT * FROM visits_APAC
) x
GROUP BY month, profile_id
ORDER BY month, profile_id;
```

### 3.4 Filtros por Data

```sql
SELECT hire_date
FROM employees
WHERE hire_date > CURRENT_DATE;
```

### 3.5 Manipulação de Colunas

```sql
ALTER TABLE Employee
ALTER COLUMN employee_name TYPE VARCHAR;

UPDATE products
SET Category = 'Unknown'
WHERE Category IS NULL;

SELECT COUNT(*)
FROM products
WHERE Category = 'Unknown';

SELECT PG_TYPEOF(category) AS data
FROM sales
LIMIT 1;
```

### 3.6 Operadores de Conjunto

* **UNION** remove duplicatas.
* **UNION ALL** é mais rápido porque **não executa DISTINCT**.

## 4. Visualização de Dados

### 4.1 Relação entre Variáveis Contínuas

* **Gráfico de dispersão**: duas variáveis contínuas, ex: horas de estudo vs pontuação.

### 4.2 Distribuição de Dados

* **Histograma**: distribuições, frequência relativa, ex: alturas de árvores.
* **Boxplot**: mediana, quartis, outliers; IQR = largura da caixa (Q3 - Q1).

### 4.3 Comparação e Agregação

* **Tabela dinâmica (pivot table)**: agrega valores numéricos em duas dimensões categóricas.
* **Gráfico de linhas**: comparar vendas por categoria ao longo do tempo.

### 4.4 Correlação e Relações Multivariadas

* **Mapa de calor (heatmap)**: visualizar correlações entre múltiplas variáveis contínuas.

### 4.5 Gráficos de Barras

* **Barras empilhadas**: valor total e contribuição de subcategorias.
* **Barras simples**: distribuição de uma variável categórica.

## 5. Resumo de Boas Práticas

1. Use **subqueries e agregações** para cálculos complexos.
2. Use **regex** para filtrar strings específicas.
3. Para **valores nulos**, utilize `UPDATE ... WHERE column IS NULL` ou `COALESCE`.
4. Escolha visualização adequada:

   * Scatter plot → duas variáveis contínuas
   * Histograma/Boxplot → distribuição
   * Gráfico de linhas → séries temporais
   * Gráfico de barras → comparação de categorias
   * Heatmap → correlação multivariada
