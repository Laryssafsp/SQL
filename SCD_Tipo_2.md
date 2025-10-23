# 🧠 Documento Técnico — SCD Tipo 2 (Slowly Changing Dimension Type 2)

## 1. Visão Geral

A **SCD Tipo 2 (Slowly Changing Dimension Type 2)** é uma técnica usada em **Data Warehousing** para **preservar o histórico completo das alterações de dados dimensionais**.  
Diferente de uma atualização simples (*overwrite*), a SCD2 **mantém versões** dos registros, permitindo identificar **o estado de um dado em um ponto no tempo**.

---

## 2. Contexto

Em um ambiente analítico, dimensões (como *Clientes*, *Produtos*, *Departamentos*, *Aprovadores*, etc.) podem mudar com o tempo.  
Por exemplo:

| ID | Departamento | Aprovador | Data de Início | Data de Fim |
|----|---------------|------------|----------------|--------------|
| 10 | Financeiro    | João       | 2024-01-01     | 2024-08-15   |
| 10 | Financeiro    | Maria      | 2024-08-16     | 9999-12-31   |

A técnica **SCD Tipo 2** garante que, mesmo após a mudança, ainda seja possível consultar relatórios de períodos passados com os dados válidos na época.

---

## 3. Princípios da SCD Tipo 2

| Etapa | Descrição |
|-------|------------|
| **1. Detectar mudanças** | Comparar os dados atuais da *stage* com a tabela de produção (dimensão). |
| **2. Fechar registros antigos** | Atualizar o registro anterior, ajustando o campo `validity_end_date` para o dia anterior à mudança. |
| **3. Inserir novo registro** | Criar uma nova versão do registro com as alterações e `validity_start_date = data atual` e `validity_end_date = '9999-12-31'`. |
| **4. Manter histórico completo** | Cada linha representa uma versão temporal válida dos dados. |

---

## 4. Campos Típicos de Controle

| Campo | Tipo | Função |
|--------|------|---------|
| `validity_start_date` | DATE | Data em que a versão do registro começou a valer. |
| `validity_end_date` | DATE | Data em que a versão deixou de valer (ou `'9999-12-31'` se ainda atual). |
| `date_upload` | DATE | Data da carga inicial. |
| `date_update` | DATE | Data da última atualização. |
| `is_current` *(opcional)* | BOOLEAN | Indica se a versão é a atual (`TRUE` / `FALSE`). |

---

## 5. Fluxo Operacional (ETL)

### 🔹 1. Extração
Os dados são extraídos de uma fonte intermediária (*staging area*) ou sistema operacional.

### 🔹 2. Transformação
Durante a comparação com a dimensão atual, cada registro é classificado como:
- **N (New):** não existe na dimensão — inserir como novo;
- **C (Change):** existe, mas com alterações — fechar o antigo e inserir nova versão;
- **E (Equal):** não mudou — nenhuma ação necessária.

### 🔹 3. Carga
- Atualiza os registros antigos (`validity_end_date = current_date - 1`);
- Insere novos registros com `validity_start_date = current_date` e `validity_end_date = '9999-12-31'`.

---

## 6. Exemplo Simplificado em SQL

```sql
-- Atualizar registros alterados
UPDATE dim_example_target AS prd
SET validity_end_date = CURRENT_DATE - INTERVAL '1 day',
    date_update = CURRENT_DATE - INTERVAL '1 day'
FROM temp_example_source AS tmp
WHERE prd.objectid = tmp.objectid
  AND prd.validity_end_date = '9999-12-31'
  AND tmp.nam_status = 'C';

-- Inserir novos e alterados
INSERT INTO dim_example_target (
    objectid, attribute_1, attribute_2,
    attribute_3, date_upload, date_update,
    validity_start_date, validity_end_date
)
SELECT objectid, attribute_1, attribute_2,
       attribute_3, CURRENT_DATE, CURRENT_DATE,
       CURRENT_DATE, '9999-12-31'
FROM temp_example_source
WHERE nam_status IN ('N', 'C');
```
---
## 7. Benefícios

✅ Mantém o histórico completo das alterações
✅ Permite auditorias e relatórios temporais
✅ Preserva integridade e rastreabilidade
✅ Compatível com ferramentas de BI e modelagem dimensional (Kimball)


---
## 8. Desvantagens

⚠️ Aumenta o volume de dados (várias versões do mesmo registro)
⚠️ Requer maior complexidade no ETL e nas consultas (filtrar apenas registros atuais, por exemplo) 


---
## 9. Boas Práticas

- Usar validity_start_date e validity_end_date sempre com intervalos consistentes (sem sobreposição).
- Definir 9999-12-31 como data padrão para registros ativos.
- Criar índices sobre objectid e validity_end_date para consultas eficientes.
- Adotar nomenclaturas padronizadas e documentar o processo de versionamento.


---
## 10. Resumo Visual

```yaml
[Tabela Dimensional - SCD Tipo 2]

objectid | atributo | validade_início | validade_fim   | status
----------|-----------|----------------|----------------|--------
1         | João      | 2024-01-01     | 2024-08-15     | fechado
1         | Maria     | 2024-08-16     | 9999-12-31     | atual

```

---
## 11. Referências

- Kimball, Ralph. The Data Warehouse Toolkit (3rd Edition)
- Inmon, Bill. Building the Data Warehouse
- Microsoft Docs: [Slowly Changing Dimensions (SCD) Types](https://learn.microsoft.com/en-us/sql/integration-services/slowly-changing-dimension)
