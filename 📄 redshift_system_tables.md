# Redshift – Tabelas de Sistema Úteis

Este documento traz uma coletânea das principais tabelas de sistema do Amazon Redshift (via `information_schema`, `pg_`, `svv_` e `stv_`) que são úteis para análise de metadados, auditoria e troubleshooting.

---

## 📑 `information_schema.columns`

Retorna informações sobre colunas em tabelas e views.


```sql
SELECT table_schema, table_name, column_name, data_type, character_maximum_length
FROM information_schema.columns
WHERE table_schema = 'public'
  AND character_maximum_length = 256;
```
📌 Uso: Identificar colunas com limite de caracteres específico ou verificar tipos de dados.

## 📚 information_schema.tables
Lista todas as tabelas e views existentes no banco.


```sql
SELECT table_schema, table_name, table_type
FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema', 'pg_catalog');
```
📌 Uso: Exploração de objetos existentes por schema.

## 🔍 svv_table_info
Exibe metadados de armazenamento para cada tabela, como número de linhas, tamanho em disco, compressão e distkey/sortkey.


```sql
SELECT *
FROM svv_table_info
WHERE schema = 'public';
```
📌 Uso: Avaliar volume de dados e eficiência de distribuição/sort.

## 🧱 svv_columns
Semelhante ao information_schema.columns, mas com informações extras sobre distribuição e sort key.


```sql
SELECT * FROM svv_columns
WHERE table_schema = 'public';
```
📌 Uso: Verificar colunas que são DISTKEY, SORTKEY, ou ENCODED.

## 📦 svv_diskusage
Traz estatísticas de uso de disco por tabela.


```sql
SELECT * FROM svv_diskusage
WHERE schema = 'public';
```
📌 Uso: Identificar tabelas grandes ou mal comprimidas.

## 🧠 pg_table_def
Exibe definição de tabelas (campos, tipos etc.).


```sql
SELECT * FROM pg_table_def
WHERE schemaname = 'public';
```
📌 Uso: Alternativa leve ao information_schema.columns para debugging rápido.

📊 stv_blocklist
Lista blocos físicos usados por tabelas – útil para verificar fragmentação.


```sql
SELECT tbl, count(*) AS num_blocks
FROM stv_blocklist
GROUP BY tbl;
```
📌 Uso: Identificar necessidade de VACUUM/ANALYZE.

## 🔧 svl_qlog
Log detalhado de queries executadas no cluster.


```sql
SELECT userid, starttime, endtime, substring, rows
FROM svl_qlog
WHERE starttime >= CURRENT_DATE - 1;
```
📌 Uso: Auditoria e performance de queries.

## 🧵 stv_recents e stv_inflight
Monitoramento em tempo real das queries em execução.


```sql
SELECT pid, user_name, start_time, query
FROM stv_recents;
```
📌 Uso: Identificar queries travadas, lentas ou em espera.

## 💬 svv_transactions
Visualiza transações abertas, úteis para debugging de locks.


```sql
SELECT *
FROM svv_transactions;
```
📌 Uso: Identificar transações travando inserts, updates, etc.

## 🔐 svv_user_privileges
Exibe permissões de usuários por objeto.


```sql
SELECT * FROM svv_user_privileges
WHERE grantee = 'seu_usuario';
```
📌 Uso: Debug de acessos e políticas de segurança.

## 🔎 svv_constraint_column_usage
Mostra quais colunas fazem parte de constraints.


```sql
SELECT * FROM svv_constraint_column_usage
WHERE table_schema = 'public';
```
📌 Uso: Entender chaves primárias e estrangeiras (embora Redshift não as aplique com enforcement real).

## 🧪 Extra: Listar colunas com valor maior que o permitido

```sql
SELECT table_schema, table_name, column_name, character_maximum_length
FROM information_schema.columns
WHERE character_maximum_length IS NOT NULL
  AND character_maximum_length < 512;
```
📌 Uso: Para validar possíveis quebras de VARCHAR (como em erros de "value too long").

📁 Observações
- svv_ são views públicas que combinam várias tabelas internas.
- stv_ são visões temporárias em tempo real.
- pg_ e information_schema são os catálogos padrão do PostgreSQL e funcionam bem no Redshift para introspecção.

🧰 Dica útil
Use o seguinte para descobrir tamanho de todas as tabelas do schema:

```sql
SELECT
  schema || '.' || "table" AS tabela,
  size AS tamanho_mb
FROM svv_table_info
WHERE schema = 'public'
ORDER BY size DESC;
```
