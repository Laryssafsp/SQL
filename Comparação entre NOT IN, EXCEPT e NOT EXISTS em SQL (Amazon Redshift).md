# Guia Técnico: Comparação entre `NOT IN`, `EXCEPT` e `NOT EXISTS` em SQL (Amazon Redshift)

Este documento descreve as diferenças, riscos, melhores práticas e recomendações para o uso de **NOT IN**, **EXCEPT** e **NOT EXISTS** em consultas SQL, com foco especial em ambientes de Data Warehouse e no Amazon Redshift.

---

## 1. Introdução

Em consultas SQL, especialmente em rotinas de ETL, governança e consistência de dados, é comum a necessidade de identificar:

* registros novos
* registros removidos
* diferenças entre conjuntos de dados

Para isso, três abordagens são frequentemente utilizadas:

* `NOT IN`
* `EXCEPT`
* `NOT EXISTS`

Embora todas possam produzir resultados semelhantes, elas possuem características distintas, que afetam:

* segurança de dados
* performance
* previsibilidade
* comportamento com valores nulos (NULL)

---

## 2. Comparação Geral

### Tabela Resumo

| Critério                          | NOT IN              | EXCEPT                | NOT EXISTS              |
| --------------------------------- | ------------------- | --------------------- | ----------------------- |
| Segurança contra NULL             | **Baixa**           | **Alta**              | **Alta**                |
| Previsibilidade                   | Média               | Alta                  | Alta                    |
| Facilidade de leitura             | Média               | Alta                  | Média                   |
| Comparação com múltiplas colunas  | Suportado via tupla | Suportado nativamente | Suportado com cláusulas |
| Performance                       | Média               | Alta                  | Alta                    |
| Indicado para detectar diferenças | Não recomendado     | **Recomendado**       | **Recomendado**         |
| Disponível no Redshift            | Sim                 | Sim                   | Sim                     |

---

## 3. Problemas do `NOT IN`

O uso de `NOT IN` é considerado a opção **menos segura**, principalmente porque valores `NULL` podem alterar completamente o resultado.

### Exemplo do problema

```sql
WHERE coluna NOT IN (1, 2, 3, NULL)
```

O resultado da condição acima será **UNKNOWN** para todas as linhas, fazendo com que:

* possivelmente **nenhum registro seja retornado**
* a lógica de negócio seja corrompida

Isso é especialmente perigoso em cargas de Data Warehouse, onde **ausência de resultado significa perda de dados**.

### Quando evitar `NOT IN`

* quando valores podem conter NULL (quase sempre)
* quando a comparação é entre conjuntos de dados grandes
* quando a operação é crítica (ex.: encerramento de registros, delta loads)

---

## 4. Vantagens do `EXCEPT`

O operador `EXCEPT` é ideal para identificar **registros novos** ou **diferenças entre conjuntos** de forma segura.

### Características

* ignora corretamente valores NULL
* compara múltiplas colunas nativamente
* é deterministicamente seguro
* performance otimizada no Redshift

### Exemplo

```sql
SELECT colunaA, colunaB
FROM tabela_atual
EXCEPT
SELECT colunaA, colunaB
FROM tabela_historico;
```

Este comando retorna **apenas o que existe em tabela_atual e não existe em tabela_historico**.

### Quando usar EXCEPT

* identificação de novos registros
* comparação simples de conjuntos
* checagem de integridade entre estágios de ETL

---

## 5. Vantagens do `NOT EXISTS`

`NOT EXISTS` é o método mais seguro para identificar registros que **deixaram de existir**, principalmente em operações de UPDATE ou DELETE.

### Exemplo

```sql
UPDATE destino t
SET dat_end_validation = CURRENT_DATE - 1
WHERE NOT EXISTS (
      SELECT 1
      FROM tabela_temporaria x
      WHERE x.id = t.id
);
```

### Características

* não sofre impactos de valores NULL
* altamente performático
* permite filtros mais complexos
* ideal para manter tabelas de exceções e auditoria

### Quando usar NOT EXISTS

* atualização de registros obsoletos
* validação de integridade entre estados atuais vs anteriores
* quando a tabela principal possui chaves compostas

---

## 6. Recomendações Práticas

### Ordem de preferência em ambiente DW (como Redshift)

1. **EXCEPT** → para identificar *novos registros*
2. **NOT EXISTS** → para identificar *registros removidos*
3. **EVITAR `NOT IN`** → somente em cenários controlados, sem NULL

### Boas práticas adicionais

* sempre validar colunas que podem conter NULL antes de usar comparação direta
* usar `DISTINCT` ao criar tabelas temporárias para evitar duplicidades
* nomear tabelas temporárias com prefixo (`#temp_`) para clareza
* preferir lógica set-based ao invés de row-by-row

---

## 7. Exemplos Práticos

### 7.1 Exemplo completo usando `EXCEPT` para identificar novos registros

```sql
CREATE TEMP TABLE tmp_novos_registros AS
SELECT
    coluna_a,
    coluna_b,
    coluna_c
FROM estado_atual
EXCEPT
SELECT
    coluna_a,
    coluna_b,
    coluna_c
FROM estado_anterior;
```

**Uso típico:** carga incremental, auditoria, detecção de inclusão.

---

### 7.2 Exemplo completo usando `NOT EXISTS` para identificar registros removidos

```sql
UPDATE destino d
SET dat_end_validation = CURRENT_DATE - 1
WHERE dat_end_validation = '9999-12-31'
  AND NOT EXISTS (
        SELECT 1
        FROM tmp_estado_atual x
        WHERE x.id = d.id
      );
```

**Uso típico:** marcação de exclusões, encerramento de vigência.

---

### 7.3 Exemplo completo comparando tuplas com `NOT IN` (não recomendado)

```sql
WHERE (col1, col2, col3) NOT IN (
      SELECT col1, col2, col3
      FROM tabela
)
```

**Risco:** se qualquer coluna contiver NULL, o resultado pode ser vazio.

---

## 8. Diagrama de Decisão (Quando Usar Qual)

```text
                      +-------------------------------+
                      |  Você precisa detectar novos? |
                      +-------------------------------+
                                   |
                                 Sim
                                   |
                                   v
                           Use → EXCEPT
                                   |
                                 Não
                                   v
      +--------------------------------------------------------------+
      | Você precisa detectar registros removidos ou inconsistentes? |
      +--------------------------------------------------------------+
                                   |
                                 Sim
                                   |
                                   v
                         Use → NOT EXISTS
                                   |
                                 Não
                                   v
              +------------------------------------------+
              | A comparação envolve possíveis valores NULL? |
              +------------------------------------------+
                             |                 |
                            Sim               Não
                             |                 |
                             v                 v
                     Evitar NOT IN      NOT IN é aceitável
```

---

## 9. Conclusão

`NOT IN`, `EXCEPT` e `NOT EXISTS` têm usos diferentes e desempenhos distintos. Em ambientes de Data Warehouse como o Amazon Redshift, o uso apropriado entre eles evita:

* comportamentos inesperados com NULL
* falhas silenciosas
* problemas de integridade de dados
* perda de performance

Uma boa escolha entre esses operadores melhora a segurança, previsibilidade e eficiência dos processos.

---

Caso deseje, posso complementar este documento com:

* exemplos avançados
* benchmarks de performance
* recomendações específicas para sua arquitetura de governança.
