# Estratégias para Escolher Entre GE e GGE em SQL

## 1. Contexto

Muitas vezes precisamos extrair o **nome do usuário** a partir de grupos diferentes, por exemplo:

* Se existir `GE`, esse deve ser o escolhido.
* Caso não exista `GE` (ou seja `NULL`), deve-se buscar `GGE`.

A seguir, duas formas de implementar essa regra.

---

## 2. Opção 1 – `COALESCE` com dois `MAX(CASE)`

```sql
COALESCE(
    MAX(CASE WHEN grupo = 'GE'  THEN nome_usuario END),
    MAX(CASE WHEN grupo = 'GGE' THEN nome_usuario END)
) AS gerente
```

### Como funciona:

1. `MAX(CASE WHEN grupo = 'GE' ...)` → tenta encontrar o valor do grupo GE.
2. Se não houver, resulta em `NULL`.
3. O `COALESCE` pega o próximo `MAX(CASE WHEN grupo = 'GGE' ...)`.
4. Assim, a prioridade lógica é **GE primeiro, depois GGE**.

✅ **Vantagem**: garante a ordem de prioridade sem depender de comparação lexicográfica.

---

## 3. Opção 2 – Um único `MAX(CASE)`

```sql
MAX(
    CASE
        WHEN grupo = 'GE'  THEN nome_usuario
        WHEN grupo = 'GGE' THEN nome_usuario
        ELSE NULL
    END
) AS gerente
```

### Como funciona:

* Verifica se o grupo é `GE`; se não for, tenta `GGE`.
* O `MAX` retorna o maior valor lexicográfico entre os resultados encontrados.

⚠️ **Risco**: se houver ao mesmo tempo `GE` e `GGE`, o banco pode devolver o valor de `GGE` caso seja lexicograficamente maior, mesmo que a regra fosse priorizar `GE`.

---

## 4. Qual usar?

* Use **Opção 1 (COALESCE)** quando a **prioridade de negócio** é clara e precisa ser garantida.
* Use **Opção 2** apenas se não houver risco de conflito entre grupos ou se a ordem lexicográfica atender ao que você precisa.

---

## 5. Resumo

* **Opção 1** → mais segura, garante a lógica de prioridade (GE > GGE).
* **Opção 2** → mais curta, mas pode dar resultados inesperados em cenários com ambos os grupos.
