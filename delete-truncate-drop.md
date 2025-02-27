# DELETE
O que faz: Remove linhas específicas de uma tabela com base em uma condição WHERE.

Uso de condições: Permite o uso de uma cláusula WHERE para remover linhas específicas.

```sql 
DELETE FROM my_table WHERE id = 1;
```
Journaling: Gera log individual para cada linha removida, o que pode afetar a performance para tabelas grandes.

Gatilhos (Triggers): Aciona gatilhos (triggers) associados à tabela.

Espaço de armazenamento: Não libera automaticamente o espaço ocupado pela tabela. O espaço deve ser recuperado com um comando VACUUM em alguns sistemas de banco de dados.

# TRUNCATE
O que faz: Remove todas as linhas de uma tabela de forma rápida e eficiente.

Uso de condições: Não permite o uso de cláusula WHERE. Remove todas as linhas de uma vez.

```sql 
TRUNCATE TABLE my_table;
```
**Journaling**: Não gera log individual para cada linha removida, o que melhora a performance.

**Gatilhos (Triggers)**: Não aciona gatilhos (triggers) associados à tabela.

**Espaço de armazenamento**: Libera automaticamente o espaço ocupado pela tabela, tornando-se mais eficiente em termos de gerenciamento de espaço.

# DROP
**O que faz**: Remove completamente uma tabela, uma base de dados ou outros objetos de banco de dados.

**Uso de condições**: Não permite o uso de cláusula WHERE. Remove o objeto inteiro.

```sql 
DROP TABLE my_table;
```
**Journaling**: Gera log para a remoção do objeto, mas não para cada linha individual.

**Gatilhos (Triggers)**: Não aciona gatilhos porque o objeto inteiro é removido.

**Espaço de armazenamento**: Libera automaticamente o espaço ocupado pelo objeto, incluindo seu esquema.

| Comando  | Descrição |
|----------|------------|
| **DELETE**  | Remove linhas específicas, permite WHERE, aciona gatilhos, pode ser mais lento para tabelas grandes devido ao journaling. |
| **TRUNCATE** | Remove todas as linhas, não permite WHERE, não aciona gatilhos, mais rápido e eficiente para limpar tabelas grandes. |
| **DROP**     | Remove o objeto inteiro (tabela, base de dados), não permite WHERE, não aciona gatilhos, libera todo o espaço do objeto. |
