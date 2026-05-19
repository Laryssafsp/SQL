# Documentação: Consultas de Tabelas de Sistema no Amazon Redshift

Este documento reúne todas as opções, views, e scripts mostrados nesta conversa para consultar metadados, tabelas internas, externas e histórico de queries no Amazon Redshift (incluindo Redshift Serverless). Os nomes utilizados são genéricos e podem ser substituídos pelos nomes reais do seu ambiente.

---

# 📌 1. Verificar se uma tabela **interna** existe

## Usando `information_schema.tables` (SQL padrão)

```sql
SELECT *
FROM information_schema.tables
WHERE table_schema = '<schema>'
  AND table_name = '<tabela>';
```

**Retorno:**

* Linha encontrada → tabela existe
* Nenhuma linha → tabela não existe

## Usando `svv_tables` (view nativa Redshift)

```sql
SELECT *
FROM svv_tables
WHERE schemaname = '<schema>'
  AND tablename = '<tabela>';
```

## Listar todas as tabelas internas de um schema

```sql
SELECT tablename
FROM svv_tables
WHERE schemaname = '<schema>'
ORDER BY tablename;
```

---

# 📌 2. Verificar tabelas **Spectrum** (tabelas externas)

Tabelas externas **não aparecem** em `information_schema.tables` ou em várias views internas.
Use sempre as views abaixo.

## Listar tabelas externas (Spectrum)

```sql
SELECT *
FROM svv_external_tables
WHERE schemaname = '<schema_externo>';
```

## Ver colunas de uma tabela externa

```sql
SELECT *
FROM svv_external_columns
WHERE schemaname = '<schema_externo>'
  AND tablename = '<tabela>';
```

## Listar schemas externos

```sql
SELECT *
FROM svv_external_schemas;
```

---

# 📌 3. Consultar histórico de queries (sys_query_history)

Colunas variam entre Redshift clássico e Serverless. Ajustar conforme o ambiente.

## Obter queries que referenciam tabelas específicas

```sql
SELECT
    q.query_id,
    q.database_name,
    q.start_time,
    q.end_time,
    qt.text AS sql_text,
    CASE
        WHEN LOWER(qt.text) LIKE 'copy %' THEN 'COPY'
        WHEN LOWER(qt.text) LIKE 'insert into%' THEN 'INSERT'
        WHEN LOWER(qt.text) LIKE 'update %' THEN 'UPDATE'
        WHEN LOWER(qt.text) LIKE 'delete %' THEN 'DELETE'
        WHEN LOWER(qt.text) LIKE 'create table as%' THEN 'CTAS'
        ELSE 'OUTRO'
    END AS tipo_operacao
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%<schema>.<tabela_1>%'
   OR LOWER(qt.text) LIKE '%<schema>.<tabela_2>%'
   OR LOWER(qt.text) LIKE '%<schema>.<tabela_3>%'
ORDER BY q.start_time DESC NULLS LAST;
```

## Ver colunas disponíveis na sys_query_history

```sql
SELECT column_name
FROM information_schema.columns
WHERE table_name = 'sys_query_history';
```

---

# 📌 4. Verificar metadados adicionais

## `pg_table_def` (somente tabelas internas)

```sql
SELECT *
FROM pg_table_def
WHERE schemaname = '<schema>'
  AND tablename = '<tabela>';
```

## Ver localização de tabelas externas no S3

```sql
SELECT schemaname, tablename, location
FROM svv_external_tables
WHERE schemaname = '<schema_externo>';
```

---

# 📌 5. Modelos genéricos de nomes

| Tipo             | Nome genérico      | Exemplos                         |
| ---------------- | ------------------ | -------------------------------- |
| Schema interno   | `<schema>`         | analytics, public, sales_data    |
| Schema externo   | `<schema_externo>` | spectrum_raw, ext_logs           |
| Tabelas internas | `<tabela>`         | clientes, vendas, tmp_workspaces |
| Tabelas Spectrum | `<tabela_externa>` | logs_raw, parquet_data           |

---

# 📌 6. Checklist para análise de origem de dados

1. **Identificar se a tabela é interna ou externa**

   * Interna → `svv_tables`
   * Externa → `svv_external_tables`

2. **Ver histórico de inserções**

   * `sys_query_history` + `sys_query_text`

3. **Identificar tipo de operação**

   * COPY
   * INSERT
   * UPDATE
   * DELETE
   * CTAS

4. **Confirmar existência**

   * Interna → `information_schema.tables`
   * Externa → `svv_external_tables`

5. **Consultar colunas**

   * Interna → `pg_table_def`
   * Externa → `svv_external_columns`

---

Se quiser, posso incluir no documento:

* fluxo completo de auditoria de tabelas
* consultas para lineage (origem/destino)
* templates para PowerBI e ETL
* diagramas visuais (caso deseje)

---

# 📌 7. Consultas para *Lineage* (Origem/Destino)

Estas consultas ajudam a identificar **de onde vêm os dados de uma tabela** e **para onde ela envia dados**, usando histórico de queries.

## ▶ 7.1. Encontrar tabelas de ORIGEM (tabelas lidas)

```sql
SELECT 
    q.query_id,
    q.start_time,
    qt.text AS sql_text
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%from <schema>.<tabela>%'
   OR LOWER(qt.text) LIKE '%join <schema>.<tabela>%'
ORDER BY q.start_time DESC;
```

Mostra todas as queries que **leram** a tabela.

## ▶ 7.2. Encontrar tabelas de DESTINO (tabelas que recebem dados)

```sql
SELECT 
    q.query_id,
    q.start_time,
    qt.text AS sql_text
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%insert into%'
  AND LOWER(qt.text) LIKE '%<schema>.<tabela>%'
ORDER BY q.start_time DESC;
```

Mostra queries que **escreveram** na tabela.

## ▶ 7.3. Encontrar COPY → origem S3

```sql
SELECT 
    q.query_id,
    q.start_time,
    qt.text
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%copy <schema>.<tabela> %'
ORDER BY q.start_time DESC;
```

Permite identificar buckets e arquivos usados no COPY.

## ▶ 7.4. Encontrar CTAS (Create Table As Select)

```sql
SELECT 
    q.query_id,
    q.start_time,
    qt.text
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%create table%as select%'
  AND LOWER(qt.text) LIKE '%<tabela>%';
```

Essas queries geralmente criam pipelines de ETL.

---

# 📌 8. Fluxo Completo de Auditoria de Tabelas

Este fluxo documenta como auditar tabelas internas e externas, sua origem e destino, e movimentação de dados.

## ▶ 8.1. Passo 1 — Verificar se a tabela existe

* Interna → `svv_tables`
* Externa → `svv_external_tables`

## ▶ 8.2. Passo 2 — Verificar estrutura

* Interna → `pg_table_def`
* Externa → `svv_external_columns`

## ▶ 8.3. Passo 3 — Capturar histórico de manipulação

```sql
SELECT q.query_id, q.start_time, qt.text
FROM sys_query_history q
JOIN sys_query_text qt ON qt.query_id = q.query_id
WHERE LOWER(qt.text) LIKE '%<tabela>%'
ORDER BY q.start_time DESC;
```

Mostra todas as queries que tocaram a tabela.

## ▶ 8.4. Passo 4 — Identificar tipos de operações

```sql
CASE
    WHEN LOWER(qt.text) LIKE 'insert into%' THEN 'INSERT'
    WHEN LOWER(qt.text) LIKE 'update %' THEN 'UPDATE'
    WHEN LOWER(qt.text) LIKE 'delete %' THEN 'DELETE'
    WHEN LOWER(qt.text) LIKE 'copy %' THEN 'COPY'
    WHEN LOWER(qt.text) LIKE 'create table as%' THEN 'CTAS'
END AS tipo_operacao
```

## ▶ 8.5. Passo 5 — Determinar origem dos dados

Use consultas de lineage de **origem**:

```sql
SELECT *
FROM sys_query_text qt
WHERE LOWER(qt.text) LIKE '%from <tabela>%'
   OR LOWER(qt.text) LIKE '%join <tabela>%';
```

## ▶ 8.6. Passo 6 — Determinar destino

Use consultas de **destino**:

```sql
SELECT *
FROM sys_query_text
WHERE LOWER(text) LIKE '%insert into <tabela>%';
```

## ▶ 8.7. Passo 7 — Verificar origem externa (S3)

```sql
SELECT location
FROM svv_external_tables
WHERE tablename = '<tabela_externa>';
```

## ▶ 8.8. Passo 8 — Criar trilha completa (resumo)

### Para cada tabela:

* Verificar se existe
* Levantar origem (FROM/JOIN)
* Levantar destino (INSERT INTO)
* Levantar COPYs (S3)
* Verificar CTAS
* Mapear dependências

No Amazon Redshift, existem várias formas de listar todas as tabelas de um banco de dados, dependendo se você quer usar SQL, metacomandos ou AWS CLI.
Segue um resumo das opções mais comuns para ver todas as tabelas (incluindo internas, externas e de todos os esquemas):

## Usando o comando SHOW TABLES (SQL nativo do Redshift)
O comando SHOW TABLES lista tabelas de um esquema específico, com opção de filtro e limite 
```
-- Lista todas as tabelas do esquema 'public'
SHOW TABLES FROM SCHEMA dev.public;


-- Lista todas as tabelas que terminam com 'view' no esquema 's1'
SHOW TABLES FROM SCHEMA dev.s1 LIKE '%view';

-- Limita a 50 resultados
SHOW TABLES FROM SCHEMA dev.public LIMIT 50;
```
Obs.: Se houver mais de 10.000 tabelas no resultado, o comando retorna erro.

## Usando a visão de sistema SVV_ALL_TABLES
Essa visão retorna todas as tabelas (internas e externas) que o usuário tem permissão para ver 
```
-- Todas as tabelas visíveis no banco
SELECT database_name, schema_name, table_name, table_type
FROM svv_all_tables
ORDER BY database_name, schema_name, table_name;

-- Você pode filtrar por esquema ou banco:

SELECT * 
FROM svv_all_tables
WHERE database_name = 'meu_banco'
  AND schema_name = 'public';
```
## Usando pg_table_def (catálogo interno)
Lista tabelas e colunas de um esquema 
```
SELECT DISTINCT schemaname, tablename
FROM pg_table_def
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY schemaname, tablename;
```
## Usando metacomando \d no RSQL
Se estiver no Query Editor v2 ou cliente compatível com RSQL 
```
\d        -- Lista tabelas e views do esquema atual
\dS       -- Inclui objetos do sistema
\d+       -- Mostra metadados adicionais
```
##  Usando AWS CLI (redshift-data list-tables)
Se preferir via terminal 
```
Copiar código
aws redshift-data list-tables \
    --cluster-identifier meu-cluster \
    --database dev \
    --schema public \
    --db-user meu_usuario
```
✅ Recomendação:
Se você quer todas as tabelas de todos os esquemas com um único comando SQL, use:
```
SELECT database_name, schema_name, table_name, table_type
FROM svv_all_tables
ORDER BY database_name, schema_name, table_name;
Isso cobre tabelas internas, externas e compartilhadas, respeitando suas permissões.
Este fluxo permite montar lineage completo SEM ferramentas externas.
```
---

