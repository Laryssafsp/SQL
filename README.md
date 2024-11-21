
# Dicas SQL

## Retornar um intervalo de dias 
```
SELECT *
FROM sua_tabela
WHERE data_coluna >= CURRENT_DATE - INTERVAL '30 days';
```

CURRENT_DATE: Retorna a data atual (sem a parte de hora).
INTERVAL '30 days': Representa um intervalo de 30 dias.
data_coluna >= CURRENT_DATE - INTERVAL '30 days': Filtra os registros onde a data na coluna data_coluna é igual ou posterior a 30 dias atrás.


## Se a data incluir hora (ou seja, for do tipo timestamp), 
```
SELECT *
FROM sua_tabela
WHERE data_coluna >= CURRENT_TIMESTAMP - INTERVAL '30 days';
```


