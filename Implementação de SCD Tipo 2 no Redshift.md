Implementação de SCD Tipo 2 no Redshift

---

### 1. Contexto

Slowly Changing Dimension Tipo 2 (SCD2) é um padrão de Data Warehouse que permite manter o histórico completo das alterações em dimensões. Este guia é aplicável a tabelas que possuem histórico diário, várias colunas, e precisam ser transformadas em uma tabela SCD2 eficiente no Redshift.

---

### 2. Estrutura Recomendada da Tabela SCD2

* `validity_start_date` → data de início da versão da linha.
* `validity_end_date` → data de fim da versão (`'9999-12-31'` para linha ativa).
* Colunas de negócio relevantes (ex.: `idt_product`, `nam_product`, `status`, etc.).
* Opcional: `dat_load` para auditoria.

> Observação: Não é necessário manter histórico diário redundante; apenas mudanças reais são registradas.

---

### 3. Fluxo Lógico para Transformar Histórico Diário em SCD2

1. **Identificar chave de negócio**: Colunas que definem a identidade de uma linha (ex.: `idt_product`).
2. **Detectar mudanças reais**: Comparar todas as colunas relevantes que definem a versão.
3. **Remover duplicatas diárias**: Manter apenas a primeira ocorrência de cada versão.
4. **Definir validade das versões**:

   * `validity_start_date` = primeira data da versão.
   * `validity_end_date` = dia anterior à próxima versão.
   * Última versão ativa → `'9999-12-31'`.

---

### 4. Implementação em Redshift (SQL Genérico)

```sql
WITH unique_versions AS (
    SELECT
        idt_product,
        idt_product_category,
        idt_domain,
        idt_macro_domain,
        nam_product,
        origin_metadata_qualified_name,
        des_definition,
        document_link,
        origin_metadata_link,
        product_status,
        status,
        dax_metadata_qualified_name,
        dax_metadata_link,
        suggested_dax_metadata,
        flg_will_be_in_looker,
        flg_published_on_looker,
        flg_commercial_hierarchy,
        observation,
        epic_link,
        idt_team,
        MIN(dat_load) AS validity_start_date
    FROM your_table
    GROUP BY
        idt_product,
        idt_product_category,
        idt_domain,
        idt_macro_domain,
        nam_product,
        origin_metadata_qualified_name,
        des_definition,
        document_link,
        origin_metadata_link,
        product_status,
        status,
        dax_metadata_qualified_name,
        dax_metadata_link,
        suggested_dax_metadata,
        flg_will_be_in_looker,
        flg_published_on_looker,
        flg_commercial_hierarchy,
        observation,
        epic_link,
        idt_team
),
versioned AS (
    SELECT
        *,
        LEAD(validity_start_date) OVER (
            PARTITION BY idt_product
            ORDER BY validity_start_date
        ) AS next_start
    FROM unique_versions
)
SELECT
    idt_product,
    idt_product_category,
    idt_domain,
    idt_macro_domain,
    nam_product,
    origin_metadata_qualified_name,
    des_definition,
    document_link,
    origin_metadata_link,
    product_status,
    status,
    dax_metadata_qualified_name,
    dax_metadata_link,
    suggested_dax_metadata,
    flg_will_be_in_looker,
    flg_published_on_looker,
    flg_commercial_hierarchy,
    observation,
    epic_link,
    idt_team,
    validity_start_date,
    CASE
        WHEN next_start IS NULL THEN '9999-12-31'::date
        ELSE DATEADD(day, -1, next_start)
    END AS validity_end_date
FROM versioned
ORDER BY idt_product, validity_start_date;
```

---

### 5. Comparação de Performance

| Método                                                                    | Descrição                                                      | Performance no Redshift                                                               |
| ------------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `MIN(dat_load) + GROUP BY`                                                | Agrupa por todas as colunas relevantes e pega a primeira carga | **Mais eficiente**; Redshift lida bem com agregações e uso de SORTKEY/DISTKEY         |
| `ROW_NUMBER() OVER(PARTITION BY todas as colunas ORDER BY dat_load DESC)` | Numera todas as linhas e filtra a última                       | Mais pesado; ordenação em memória de todas as linhas, alto custo com histórico grande |

**Conclusão**: usar `GROUP BY + MIN()` é recomendado para tabelas históricas grandes.

---

### 6. Inserção/Atualização em Tabela SCD2 Final

Fluxo típico:

1. Comparar staging com tabela SCD2 existente.
2. Atualizar `validity_end_date` das versões antigas (`current_date - 1`) se houver mudança.
3. Inserir novas versões com `validity_start_date = current_date` e `validity_end_date = '9999-12-31'`.

---

### 7. Boas Práticas

* Usar **DISTKEY** na chave de negócio (`idt_product`) e **SORTKEY** em `validity_start_date` para acelerar consultas e joins.
* Criar **hash** de colunas de versão para comparar mudanças em vez de muitas colunas individualmente.
* Evitar armazenar histórico diário redundante; manter apenas mudanças reais.
* Monitorar volume de dados e considerar particionamento se a tabela for muito grande.

---

