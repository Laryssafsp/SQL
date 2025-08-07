## Comparações de Data e Hora

```sql
DELETE FROM database.table
WHERE column < CURRENT_DATE - INTERVAL '2 years';
```

Essa cmparação funciona corretamente com colunas TIMESTAMP, pois o CURRENT_DATE é automaticamente convertido para o mesmo tipo durante a comparação. <br>
Se for necessário incluir a hora como parte das comparações, você pode usar CURRENT_TIMESTAMP:


## Inserções de Dados
```sql
SELECT column::timestamp
FROM self_service_analytics_spectrum.capacity_unit_details;
```


## Funções de Manipulação e Truncamento
```sql
DELETE FROM database.table
WHERE DATE_TRUNC('day', column) = CURRENT_DATE;
```

## UTC 
```sql
CONVERT_TIMEZONE('UTC', 'America/Sao_Paulo', SYSDATE) AS dat_load
```

## Intervao de Datas
```sql
WHERE dat_date >= CURRENT_DATE - 30 
  AND dat_date < CURRENT_DATE
```
```sql
WHERE dat_date BETWEEN CURRENT_DATE - 30 AND CURRENT_DATE - 1

```

## 
```sql

```
