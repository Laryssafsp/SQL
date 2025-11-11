# Documentação: Diferença entre Filtrar no JOIN e Filtrar no WHERE

Esta documentação explica a diferença entre aplicar filtros na cláusula **JOIN** e aplicar filtros na cláusula **WHERE**, usando nomes de tabelas, colunas e schemas genéricos. Também inclui exemplos visuais de tabelas e queries para facilitar o entendimento.

---

## 1. Estrutura das Tabelas de Exemplo

### Tabela A: `schema1.tabela_a`

Representa a tabela principal.

| id_a | nome_schema |
| ---- | ----------- |
| 1    | alpha       |
| 2    | beta        |
| 3    | gamma       |
| 4    | delta       |

---

### Tabela B: `schema2.tabela_b`

Representa a tabela complementar.

| id_b | schema_ref | categoria |
| ---- | ---------- | --------- |
| 10   | alpha      | X         |
| 11   | beta       | Y         |
| 12   | beta       | X         |
| 13   | epsilon    | X         |

Observações:

* Nem todos os valores de `nome_schema` da Tabela A existem na Tabela B.
* Alguns valores da Tabela B não existem na Tabela A.

---

## 2. Objetivo da Demonstração

Mostrar o que acontece quando aplicamos **filtros**:

1. Na cláusula **WHERE**
2. Dentro da cláusula **JOIN**

Usando um `LEFT JOIN`.

---

# 3. Diferença Fundamental

## ✅ 3.1. Filtro no WHERE pode “anular” o LEFT JOIN

Quando o filtro usa **colunas da tabela da direita (Tabela B)** na cláusula **WHERE**, o `LEFT JOIN` deixa de funcionar como esperado.

Isso acontece porque, quando não existe correspondência na Tabela B, o `LEFT JOIN` produz valores **NULL**. E um filtro como:

```sql
WHERE tabela_b.categoria = 'X'
```

eliminação automaticamente todas as linhas onde `categoria` é `NULL`, ou seja, **todas as linhas sem correspondência**, transformando o LEFT JOIN em um comportamento parecido com um INNER JOIN.

---

## ✅ Exemplo 1: Filtro no WHERE (comportamento indesejado)

```sql
SELECT a.*, b.categoria
FROM schema1.tabela_a a
LEFT JOIN schema2.tabela_b b
    ON a.nome_schema = b.schema_ref
WHERE b.categoria = 'X';
```

### ❌ Resultado:

* Retorna **somente linhas de A que possuem registros em B com categoria X**.
* As linhas de A que **não possuem correspondência em B** são **eliminadas**.

### Linhas retornadas:

| id_a | nome_schema | categoria |
| ---- | ----------- | --------- |
| 1    | alpha       | X         |
| 2    | beta        | X         |

### Linhas que foram perdidas:

* id_a = 3 (gamma) — não existe em B
* id_a = 4 (delta) — não existe em B

---

## ✅ 3.2. Filtro dentro do JOIN preserva o LEFT JOIN

Agora movemos o filtro referente à Tabela B para a cláusula **ON**, mantendo o comportamento correto do LEFT JOIN.

### Exemplo 2: Filtro no JOIN (comportamento correto)

```sql
SELECT a.*, b.categoria
FROM schema1.tabela_a a
LEFT JOIN schema2.tabela_b b
    ON a.nome_schema = b.schema_ref
    AND b.categoria = 'X';
```

### ✅ Resultado:

* **Todas as linhas da Tabela A são retornadas**.
* Apenas os registros da Tabela B são filtrados pela categoria X.
* Onde não há correspondência, os valores da tabela B permanecem NULL.

### Linhas retornadas:

| id_a | nome_schema | categoria |
| ---- | ----------- | --------- |
| 1    | alpha       | X         |
| 2    | beta        | X         |
| 3    | gamma       | NULL      |
| 4    | delta       | NULL      |

---

# 4. Comparação Visual

### ❌ Filtro no WHERE (STOP — remove NULLs)

```
LEFT JOIN
WHERE tabela_b.coluna = valor
→ Remove linhas que não possuem correspondência
→ Comportamento semelhante a INNER JOIN
```

### ✅ Filtro no JOIN (PASS — preserva NULLs)

```
LEFT JOIN (com filtro)
WHERE apenas colunas da tabela A
→ Mantém todas as linhas da esquerda
```

---

# 5. Regra Prática

### ✅ Use **WHERE** somente para filtrar colunas da tabela principal (A).

### ✅ Use **JOIN ... ON** para filtrar colunas da tabela da direita (B).

**Regra de ouro:**

> *Se você quer manter todas as linhas da tabela esquerda, nunca filtre colunas da tabela da direita no WHERE.*

---

# 6. Conclusão

Mover o filtro:

* **Do WHERE → para o JOIN**

Garante que o `LEFT JOIN` funcione corretamente, mantendo todas as linhas da tabela principal.

Essa prática é recomendada para qualquer situação em que você deseja preservar os registros da tabela da esquerda, mesmo quando não há correspondências na tabela da direita.

---
