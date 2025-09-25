# O que é Valor Lexicográfico em SQL

## 1. Conceito

Valor **lexicográfico** é a forma como os bancos de dados comparam valores de texto (strings) quando utilizam funções de agregação como `MIN()` e `MAX()`. Essa comparação segue a **ordem alfabética** (ou ordem de código dos caracteres), semelhante a como as palavras são organizadas em um dicionário.

## 2. Exemplos Simples

### Exemplo 1 – Usando `MAX`

```sql
SELECT MAX(nome) AS maior_nome
FROM (VALUES ('Ana'), ('Carlos'), ('Pedro')) AS t(nome);
```

**Resultado:** `Pedro`
Porque `P` vem depois de `C` e `A` na ordem alfabética.

### Exemplo 2 – Usando `MIN`

```sql
SELECT MIN(nome) AS menor_nome
FROM (VALUES ('Ana'), ('Carlos'), ('Pedro')) AS t(nome);
```

**Resultado:** `Ana`
Porque `A` é o menor valor lexicográfico.

---

## 3. Implicação em Consultas

Quando você faz:

```sql
MAX(CASE WHEN grupo = 'GE' THEN nome_usuario
         WHEN grupo = 'GGE' THEN nome_usuario END) AS gerente
```

Se houver registros com `GE` e `GGE` ao mesmo tempo, o banco não vai "saber" que você queria priorizar o `GE`. Ele vai simplesmente devolver o **maior em ordem alfabética**.

Exemplo:

* GE → `Ana`
* GGE → `Pedro`

O resultado seria `Pedro`, mesmo que a regra de negócio fosse dar prioridade ao `GE`.

---

## 4. Como Resolver

Para garantir prioridade, use `COALESCE` com dois `MAX(CASE)` separados:

```sql
COALESCE(
    MAX(CASE WHEN grupo = 'GE'  THEN nome_usuario END),
    MAX(CASE WHEN grupo = 'GGE' THEN nome_usuario END)
) AS gerente
```

Aqui a lógica funciona assim:

1. Primeiro tenta encontrar alguém do grupo `GE`.
2. Se não encontrar (`NULL`), retorna o valor de `GGE`.

---

## 5. Resumo

* **Lexicográfico** = comparação alfabética.
* `MAX` e `MIN` em textos usam essa regra.
* Cuidado: isso pode causar resultados inesperados se você queria dar **prioridade lógica**.
* Para evitar problemas, combine `CASE` + `COALESCE` ou crie regras explícitas de prioridade.
