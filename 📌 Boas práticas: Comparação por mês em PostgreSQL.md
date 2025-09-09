# 📌 Boas práticas: Comparação por mês em PostgreSQL

## 1. Problema comum
Quando trabalhamos com colunas `DATE` ou `TIMESTAMP` e queremos comparar **meses inteiros**, muitas vezes aplicamos `DATE_TRUNC('month', ...)`.  
Se o filtro do `WHERE` não estiver **alinhado ao nível de truncamento**, os resultados podem:  
- retornar **menos meses** que o esperado,  
- incluir o **mês atual** indevidamente,  
- ou até **excluir meses inteiros** caso os registros sejam parciais.  

---

## 2. Padrão recomendado

Sempre normalize o intervalo de tempo para **primeiro dia do mês** antes de comparar.  

✅ Exemplo — últimos 6 meses completos (excluindo mês atual):  

```sql
-- Últimos 6 meses no calendário
WITH ultimos_6_meses AS (
    SELECT
        DATE_TRUNC('month', data_calendario)::DATE AS mes
    FROM tabela_calendario
    WHERE data_calendario >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
      AND data_calendario <  DATE_TRUNC('month', CURRENT_DATE)
    GROUP BY 1
)
-- Meses com registros
, meses_com_registro AS (
    SELECT
        DISTINCT DATE_TRUNC('month', data_ref)::DATE AS mes
    FROM tabela_fatos
    WHERE data_ref >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
      AND data_ref <  DATE_TRUNC('month', CURRENT_DATE)
)
-- Comparação
SELECT c.mes,
       COUNT(r.mes) AS registros
FROM ultimos_6_meses c
LEFT JOIN meses_com_registro r
       ON c.mes = r.mes
GROUP BY c.mes
ORDER BY c.mes;
```

---

## 3. Por que funciona
- `DATE_TRUNC('month', CURRENT_DATE)` → garante corte sempre no **primeiro dia do mês atual (00:00:00)**.  
- Intervalo `>= início` e `< início do mês atual` → inclui meses completos, mas nunca o mês em andamento.  
- `DATE_TRUNC` aplicado tanto no calendário quanto na tabela de fatos → garante que o join sempre compare meses alinhados.  

---

## 4. Checklist rápido
- [ ] Sempre truncar ambos os lados (`calendar` e `fato`) para `month`.  
- [ ] Usar intervalo **fechado no início** (`>=`) e **aberto no fim** (`<`).  
- [ ] Evitar usar `-7` ou `-1` meses direto sem truncar — pode cortar meses no meio.  

---

## ⚡ Performance em filtros mensais (PostgreSQL)

### 1. Cenário
Quando precisamos consolidar dados **por mês**, temos duas abordagens comuns no `WHERE`:

#### A) Aplicando `DATE_TRUNC` na coluna
```sql
SELECT DATE_TRUNC('month', data_ref)::DATE AS mes
FROM tabela_fatos
WHERE DATE_TRUNC('month', data_ref) >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
  AND DATE_TRUNC('month', data_ref) <  DATE_TRUNC('month', CURRENT_DATE);
```
- ✅ Simples e legível.  
- ❌ Menos performático: força conversão (`DATE_TRUNC`) em **todas as linhas** antes da comparação → pode impedir uso de **índices**.  

#### B) Comparando intervalo de datas cruas
```sql
SELECT DATE_TRUNC('month', data_ref)::DATE AS mes
FROM tabela_fatos
WHERE data_ref >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
  AND data_ref <  DATE_TRUNC('month', CURRENT_DATE);
```
- ✅ Mais performático: a condição atua diretamente sobre a coluna `data_ref`.  
- ✅ Pode usar índice em `data_ref`.  
- ❌ Um pouco menos intuitivo para leitura.  

---

### 2. Comparação prática
- Se a coluna é **grande (muitos registros)** → prefira **comparação direta** (`WHERE data_ref >= ... AND data_ref < ...`).  
- Se a consulta é **ad hoc ou sem índice** → usar `DATE_TRUNC` no `WHERE` não pesa tanto.  

---

### 3. Padrão recomendado
**Sempre filtrar por datas cruas no `WHERE` e aplicar `DATE_TRUNC` apenas no `SELECT` ou no `GROUP BY`.**  

✅ Exemplo consolidado (6 meses completos, excluindo mês atual):  

```sql
WITH meses_calendario AS (
    SELECT DATE_TRUNC('month', data_calendario)::DATE AS mes
    FROM tabela_calendario
    WHERE data_calendario >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
      AND data_calendario <  DATE_TRUNC('month', CURRENT_DATE)
    GROUP BY 1
)
, meses_fatos AS (
    SELECT DISTINCT DATE_TRUNC('month', data_ref)::DATE AS mes
    FROM tabela_fatos
    WHERE data_ref >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
      AND data_ref <  DATE_TRUNC('month', CURRENT_DATE)
)
SELECT c.mes,
       COUNT(f.mes) AS registros
FROM meses_calendario c
LEFT JOIN meses_fatos f
       ON c.mes = f.mes
GROUP BY c.mes
ORDER BY c.mes;
```

---

📎 **Nota interna:**  
O uso de `DATE_TRUNC` dentro do `WHERE` deve ser evitado em tabelas de fatos muito volumosas, pois pode inviabilizar o uso de índices.  
Padronizar sempre em **comparação direta** garante performance e consistência.

## Escolha entre `INTERVAL` e `ADD_MONTHS`

Existem duas formas comuns de calcular datas relativas em filtros mensais:

- **Usando `CURRENT_DATE - INTERVAL '6 months'`**
  - Mais legível
  - Padrão SQL ANSI
  - Recomendado no PostgreSQL para consistência e clareza

- **Usando `ADD_MONTHS(CURRENT_DATE, -6)`**
  - Mais comum em ambientes que imitam Oracle ou Redshift
  - Útil para times que já padronizaram este estilo
  - Precisa de cuidado para sempre alinhar com `DATE_TRUNC('month', ...)`

### Exemplo equivalente

```sql
-- Forma recomendada (PostgreSQL puro)
WHERE dat_reference >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '6 months')
  AND dat_reference <  DATE_TRUNC('month', CURRENT_DATE)

-- Forma alternativa (compatibilidade Oracle/Redshift)
WHERE dat_reference >= DATE_TRUNC('month', ADD_MONTHS(CURRENT_DATE, -6))
  AND dat_reference <  DATE_TRUNC('month', CURRENT_DATE)
```
