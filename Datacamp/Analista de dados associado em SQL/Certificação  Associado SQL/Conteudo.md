# Guia de Funções SQL para Estudo (PostgreSQL)

Este documento reúne funções essenciais de **estatística**, **manipulação de dados**, **agregação**, **janela**, **expressões regulares**, **datas**, **conversão**, **COALESCE**, **joins**, e outros recursos usados frequentemente em análises SQL.

Todos os exemplos usam tabelas e colunas genéricas.

---

## 📌 1. Estatística Descritiva

### **AVG() – Média**
```sql
SELECT AVG(valor) AS media
FROM tabela;
```

### **STDDEV(), STDDEV_POP(), STDDEV_SAMP() – Desvio padrão**
```sql
SELECT STDDEV(valor) AS desvio_padrao
FROM tabela;
```

### **VARIANCE() – Variância**
```sql
SELECT VARIANCE(valor) AS variancia
FROM tabela;
```

### **PERCENTILE_CONT() – Mediana (e outros percentis)**
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY valor ASC) AS mediana
FROM tabela;
```

### **Coeficiente de Pearson (Skewness simplificado)**
```sql
WITH s AS (
  SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY valor) AS median,
    AVG(valor) AS mean,
    STDDEV(valor) AS std
  FROM tabela
)
SELECT (3 * (mean - median)) / std AS pearson_skew
FROM s;
```

---

## 📌 2. Manipulação de Datas

### **EXTRACT() – Extrair partes de datas**
```sql
SELECT EXTRACT(MONTH FROM data) AS mes FROM tabela;
```

### **DATE_TRUNC() – Truncar datas**
```sql
SELECT DATE_TRUNC('hour', data_hora) AS hora_truncada FROM tabela;
```

### **NOW() e CURRENT_DATE**
```sql
SELECT * FROM tabela WHERE data > CURRENT_DATE;
```

### **FILTRAR intervalo de datas**
```sql
WHERE data BETWEEN '2024-01-01' AND '2024-01-31'
```

---

## 📌 3. Expressões Regulares (REGEXP)

### **Encontrar valores com números**
```sql
WHERE nome ~ '[0-9]'
```

### **Começa com número**
```sql
WHERE codigo ~ '^[0-9]'
```

### **Remover números**
```sql
UPDATE tabela
SET nome = REGEXP_REPLACE(nome, '[0-9]', '', 'g');
```

---

## 📌 4. Conversões de Tipo

### **CAST e ::**
```sql
SELECT valor::numeric(10,2) FROM tabela;
```

### **pg_typeof() – Verificar tipo de dados**
```sql
SELECT pg_typeof(coluna) FROM tabela;
```

---

## 📌 5. Funções de Agregação por Grupo

### **GROUP BY com média e mediana**
```sql
SELECT
  categoria,
  AVG(preco) AS media,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY preco) AS mediana
FROM produtos
GROUP BY categoria;
```

### **Filtrar grupos com HAVING**
```sql
HAVING SUM(vendas) < 600
```

---

## 📌 6. COALESCE – Substituir valores NULL

### **Usar valor padrão**
```sql
SELECT COALESCE(nota, 0) FROM tabela;
```

### **Usar a média quando NULL**
```sql
COALESCE(nota, (SELECT AVG(nota) FROM tabela))
```

---

## 📌 7. JOINs Essenciais

### **JOIN básico**
```sql
SELECT *
FROM a
JOIN b ON a.id = b.id;
```

### **JOIN com USING (remove colunas duplicadas)**
```sql
SELECT *
FROM a
JOIN b USING (id);
```

### **LEFT JOIN**
```sql
SELECT *
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id;
```

---

## 📌 8. Subconsultas (Subqueries)

### **Subconsulta na cláusula SELECT**
```sql
SELECT
  nome,
  (SELECT COUNT(*) FROM pedidos p WHERE p.cliente_id = c.id) AS total_pedidos
FROM clientes c;
```

### **Subconsulta no WHERE com EXISTS**
```sql
SELECT *
FROM produtos p
WHERE EXISTS (
  SELECT 1
  FROM vendas v
  WHERE v.produto_id = p.id
);
```

---

## 📌 9. CTE (Common Table Expressions)

### **Usando WITH**
```sql
WITH resumo AS (
  SELECT categoria, SUM(vendas) AS total
  FROM produtos
  GROUP BY categoria
)
SELECT * FROM resumo;
```

---

## 📌 10. Funções de Janela (Window Functions)

### **ROW_NUMBER()**
```sql
SELECT nome, ROW_NUMBER() OVER (ORDER BY vendas DESC) AS posicao
FROM vendedores;
```

### **LAG() e LEAD()**
```sql
SELECT
  data,
  vendas,
  LAG(vendas) OVER (ORDER BY data) AS dia_anterior
FROM diario;
```

### **AVG() OVER() – Média móvel**
```sql
SELECT
  data,
  valor,
  AVG(valor) OVER (ORDER BY data ROWS 6 PRECEDING) AS media_7dias
FROM tabela;
```

---

## 📌 11. Arredondamento

### **ROUND()**
```sql
SELECT ROUND(preco, 2) FROM tabela;
```

---

## 📌 12. Outras Funções Úteis

### **LENGTH() – Tamanho de string**
```sql
SELECT LENGTH(nome) FROM tabela;
```

### **UPPER() e LOWER()**
```sql
SELECT UPPER(nome), LOWER(nome) FROM tabela;
```

### **CONCAT()**
```sql
SELECT CONCAT(nome, ' ', sobrenome) FROM clientes;
```

---


