# SQL POSTGRE - REDSHIFT


# Índice

1. [Calcular a diferença em minutos](#calcular-a-diferença-em-minutos)
2. [Consultar os dias da semana](#consultar-os-dias-da-semana)

---

## Calcular a diferença em minutos:

```sql
SELECT 
    ROUND(DATEDIFF(second, starttime, endtime) / 60.0, 2) AS Tempo_Execucao_min
FROM 
    dim_refresh_history;
```

---

## Consultar os dias da semana

**EXTRACT(DOW FROM data)**: A função EXTRACT com o especificador DOW retorna o dia da semana como um número inteiro:

- 0 = Domingo,
- 1 = Segunda-feira,
- 2 = Terça-feira, e assim por diante.

**CASE**: O CASE é utilizado para mapear esses números (0 a 6) para os nomes dos dias da semana em português.

```sql
SELECT 
    data,
    CASE EXTRACT(DOW FROM data)
        WHEN 0 THEN 'Domingo'
        WHEN 1 THEN 'Segunda-feira'
        WHEN 2 THEN 'Terça-feira'
        WHEN 3 THEN 'Quarta-feira'
        WHEN 4 THEN 'Quinta-feira'
        WHEN 5 THEN 'Sexta-feira'
        WHEN 6 THEN 'Sábado'
    END AS dia_da_semana
FROM 
    dias_da_semana;
```
