# Tabelas de Sistema do Amazon Redshift

As tabelas de sistema do Amazon Redshift armazenam metadados e informações de auditoria sobre consultas, sessões, carga de dados, erros e desempenho. Elas são fundamentais para monitoramento, troubleshooting e tuning do cluster.

---

## 🗂️ Tipos de Tabelas de Sistema

O Redshift possui dois tipos principais de objetos para metadados e logs:

| Tipo | Prefixo | Descrição                                                                                                                                            |
| ---- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| STL  | `stl_`  | Tabelas que registram histórico de execução (queries, erros, commits, etc.). São **transientes** — os dados antigos são sobrescritos periodicamente. |
| SVL  | `svl_`  | **Views** baseadas nas tabelas `stl_`, geralmente com joins úteis para análise.                                                                      |
| STV  | `stv_`  | Tabelas **voláteis**, que mostram o estado **atual** do cluster (sessões ativas, bloqueios, etc.).                                                   |
| PG   | `pg_`   | Tabelas compatíveis com PostgreSQL, contendo metadados de esquema, tabelas, colunas e usuários.                                                      |

---

## 🔍 Tabelas STL Mais Utilizadas

### 1. `stl_query`

Registra todas as queries executadas no cluster.

```sql
SELECT userid, query, starttime, endtime, querytxt, aborted
FROM stl_query
ORDER BY starttime DESC;
```

**Campos úteis:**

* `userid`: ID do usuário que executou a query
* `querytxt`: texto SQL completo
* `starttime` / `endtime`: tempos de execução
* `aborted`: indica se foi cancelada

---

### 2. `stl_querytext`

Contém o texto completo de cada comando SQL, útil quando `stl_query` é truncado.

```sql
SELECT q.query, t.sequence, t.text
FROM stl_querytext t
JOIN stl_query q ON q.query = t.query
ORDER BY q.starttime DESC;
```

---

### 3. `stl_utilitytext`

Armazena comandos de utilitários como `COPY`, `UNLOAD`, `VACUUM`, `ANALYZE` e `CALL` (procedures).

```sql
SELECT *
FROM stl_utilitytext
WHERE text ILIKE '%CALL%'
ORDER BY starttime DESC;
```

---

### 4. `stl_error`

Registra erros gerados durante a execução de queries.

```sql
SELECT userid, query, starttime, err_reason
FROM stl_error
ORDER BY starttime DESC;
```

---

### 5. `stl_load_errors`

Mostra erros em operações `COPY` (importação de dados).

```sql
SELECT starttime, filename, line_number, err_reason
FROM stl_load_errors
ORDER BY starttime DESC;
```

---

### 6. `stl_wlm_query`

Relaciona queries com filas WLM (Workload Management).

```sql
SELECT service_class, total_exec_time, rows, query
FROM stl_wlm_query
ORDER BY total_exec_time DESC;
```

---

## 🧠 Views SVL / STV Comuns

| View                | Descrição                                                                   |
| ------------------- | --------------------------------------------------------------------------- |
| `svl_qlog`          | Combina dados de `stl_query` e `stl_wlm_query` para análise de performance. |
| `svl_statementtext` | Mostra o SQL completo de cada statement.                                    |
| `stv_recents`       | Queries que estão **rodando agora**.                                        |
| `stv_blocklist`     | Mostra sessões bloqueadas por outras transações.                            |
| `stv_sessions`      | Lista todas as sessões conectadas ao cluster.                               |

---

## ⚙️ Exemplo: Consultar Logs de Procedures

Para ver execuções de *stored procedures*:

```sql
SELECT q.query, q.starttime, u.text
FROM stl_query q
JOIN stl_utilitytext u USING (userid, xid)
WHERE u.text ILIKE '%CALL%'
ORDER BY q.starttime DESC;
```

Se estiver usando **Redshift Serverless** ou **Data API**, esses logs podem não aparecer nas tabelas STL — nesse caso, verifique o **CloudWatch Logs**.

---

## 🧹 Retenção e Limpeza

As tabelas `stl_` são armazenadas em disco local do cluster e **não persistem indefinidamente**.

* Retenção média: ~2 a 5 dias (dependendo do tamanho do cluster e volume de queries)
* Para auditoria de longo prazo, exporte os logs periodicamente para S3:

  ```sql
  UNLOAD ('SELECT * FROM stl_query')
  TO 's3://meu-bucket/auditoria/stl_query_'
  IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftS3Role'
  CSV;
  ```

---

## 📚 Referências Oficiais

* [Amazon Redshift System Tables and Views](https://docs.aws.amazon.com/redshift/latest/dg/cm_chap_system-tables-and-views.html)
* [Monitoring Queries in Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/dg/c_the-syslog.html)
* [Redshift Serverless Query Logs](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-query-logging.html)
