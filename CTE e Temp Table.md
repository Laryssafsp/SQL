## Tabela temporária
- Materializa a tabela: Uma tabela temporária cria uma estrutura de tabela completa, que pode ser manipulada como uma tabela normal, mas com um escopo limitado à sessão ou à transação em que foi criada.
- Persistência: Ela persiste durante a sessão em que foi criada. Ou seja, uma tabela temporária só é visível para a sessão que a criou e é automaticamente descartada ao final dessa sessão ou transação, a não ser que seja explicitamente removida.
- Criação e Exclusão:
-  Para criar a tabela temporária, usamos o comando CREATE TEMP TABLE.
-  Para excluir a tabela temporária, podemos usar o comando DROP ou ela será removida automaticamente ao final da sessão.
- Escopo: Tabelas temporárias são visíveis apenas na sessão atual e não podem ser acessadas por outras sessões, o que as torna ideais para armazenar dados temporários enquanto se executa um conjunto de operações em uma única sessão.
  
```sql
CREATE TEMP TABLE temp_table AS
SELECT *, 
       NULL::VARCHAR(255) AS nam_column
FROM table;
```

```sql
DROP TABLE IF EXISTS temp_table;
```

## CTE - Common Table Expression
- Materialização temporária: Uma CTE é uma tabela temporária, mas não materializada fisicamente. Ela existe apenas durante a execução da consulta e não persiste após a execução. A CTE é mais como uma subconsulta nomeada que pode ser referenciada ao longo da consulta principal.
- Escopo e Visibilidade: A CTE é visível somente dentro da consulta em que é definida. Ela é criada apenas para aquela execução e não armazena dados fisicamente, sendo descartada automaticamente ao final da consulta.
- Criação:
-  Para criar uma CTE, usamos a palavra-chave WITH seguida do nome da CTE e da definição da consulta que a preenche.

- Exemplo de Criação:
  
```sql
with cte_table AS
SELECT *, 
       NULL::VARCHAR(255) AS nam_column
FROM table;
```

```sql
WITH cte_table AS (
  SELECT *, 
         NULL AS nam_column
  FROM table
)
SELECT * FROM cte_table;

```
## Diferenças principais entre Tabelas Temporárias e CTEs

| Característica                | Tabela Temporária                        | CTE (Common Table Expression)         |
|-------------------------------|------------------------------------------|---------------------------------------|
| **Persistência**               | Existe até o final da sessão ou transação. | Apenas durante a execução da consulta.|
| **Materialização**             | Tabela física, materializa os dados no disco. | Não materializa, apenas mantém os dados na memória durante a consulta. |
| **Visibilidade**               | Visível apenas na sessão atual.         | Visível apenas na consulta que a utiliza. |
| **Uso**                        | Ideal para armazenar dados temporários durante múltiplas operações. | Ideal para simplificar consultas complexas e evitar subconsultas repetidas. |
| **Desempenho**                 | Pode ser mais eficiente para manipulação de grandes volumes de dados. | Menos eficiente com grandes volumes de dados, já que os dados podem ser recalculados. |
| **Manipulação de Dados**       | Pode ser modificada (INSERT, UPDATE, DELETE). | Somente leitura, não pode ser modificada diretamente. |



## Conclusão
- Tabelas Temporárias são ideais quando você precisa de uma tabela de dados temporários com a capacidade de realizar operações de leitura e escrita durante uma sessão. Elas são eficazes quando você precisa de persistência temporária de dados para processamentos complexos.

- CTEs, por outro lado, são mais adequadas quando você precisa apenas de uma expressão de consulta reutilizável dentro de uma única execução. Elas são mais leves, pois não materializam dados fisicamente e são descartadas automaticamente após a execução.

