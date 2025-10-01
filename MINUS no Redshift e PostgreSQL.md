# MINUS no Redshift e PostgreSQL

## 1. Contexto

O operador `MINUS` é tradicionalmente usado em bancos como **Oracle**
para retornar registros do primeiro conjunto que **não aparecem** no
segundo.

Exemplo em Oracle:

``` sql
SELECT coluna1 FROM tabela_a
MINUS
SELECT coluna1 FROM tabela_b;
```

------------------------------------------------------------------------

## 2. Situação no PostgreSQL

No PostgreSQL, o operador equivalente é **`EXCEPT`**.\
Ou seja, a forma correta é:

``` sql
SELECT coluna1 FROM tabela_a
EXCEPT
SELECT coluna1 FROM tabela_b;
```

O PostgreSQL **não implementa oficialmente** `MINUS`.\
Se algum ambiente aceita `MINUS`, é provável que haja **extensões de
compatibilidade** ou camadas intermediárias que traduzem `MINUS` →
`EXCEPT`.

------------------------------------------------------------------------

## 3. Situação no Redshift

Historicamente, o Redshift também documenta apenas o operador
**`EXCEPT`**.\
No entanto, em alguns clusters, `MINUS` funciona como **sinônimo de
`EXCEPT`**, permitindo consultas no formato Oracle.

Exemplo válido em alguns ambientes Redshift:

``` sql
SELECT coluna1 FROM tabela_a
MINUS
SELECT coluna1 FROM tabela_b;
```

Isso pode ocorrer por: - Atualizações recentes do Redshift que
adicionaram compatibilidade (ainda não totalmente documentadas).\
- Modos de compatibilidade SQL habilitados pela AWS.\
- Ferramentas/drivers que fazem o parser e convertem `MINUS` em
`EXCEPT`.

------------------------------------------------------------------------

## 4. Resumo

-   **PostgreSQL** → use sempre `EXCEPT`.\
-   **Redshift** → oficialmente só `EXCEPT`, mas alguns clusters já
    aceitam `MINUS` como atalho.\
-   **Oracle** → usa `MINUS` nativamente.

⚠️ Recomenda-se **usar `EXCEPT`** em PostgreSQL/Redshift para evitar
problemas de portabilidade.
