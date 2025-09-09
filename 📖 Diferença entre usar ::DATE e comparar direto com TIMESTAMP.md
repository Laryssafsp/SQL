
# 📖 Diferença entre usar `::DATE` e comparar direto com `TIMESTAMP`

Este documento foi criado para consulta interna sobre boas práticas de filtragem de datas no PostgreSQL.

---

## 1. Usando `dat_creation::DATE`
```sql
WHERE dat_creation::DATE BETWEEN CURRENT_DATE - 15 AND CURRENT_DATE - 1
```
- Converte o valor de cada linha para **apenas data**.  
- Inclui todos os registros de cada dia, independentemente da hora.  
- ✅ Bom para **agregações por dia**.  
- ❌ Pode perder performance em tabelas grandes (não usa índice do timestamp).

---

## 2. Usando `dat_creation` direto
```sql
WHERE dat_creation BETWEEN CURRENT_DATE - 15 AND CURRENT_DATE - 1
```
- Faz comparação de **timestamp**.  
- `CURRENT_DATE - 1` = meia-noite do último dia (ex.: `2025-09-08 00:00:00`).  
- Só pega registros **até a meia-noite**, descartando o restante do dia.  
- Resultado: último dia do intervalo fica vazio.

---

## 3. Forma correta para incluir o último dia inteiro
```sql
WHERE dat_creation >= CURRENT_DATE - 15
  AND dat_creation < CURRENT_DATE
```
- Intervalo **[2025-08-25 00:00:00, 2025-09-09 00:00:00)**.  
- Inclui todos os horários até o fim do dia 08/09.  
- ✅ Melhor prática para trabalhar com `timestamp` (mantém índices, evita cast).  

---

## 4. Entendendo `CURRENT_DATE`, `NOW()` e `CURRENT_TIMESTAMP`

- `CURRENT_DATE` → retorna **somente a data**, sem hora.  
  - Quando usado contra `timestamp`, o Postgres converte para o início do dia (`00:00:00`).  
  - Exemplo:
    ```sql
    SELECT CURRENT_DATE;          -- 2025-09-09
    SELECT CURRENT_DATE::TIMESTAMP; -- 2025-09-09 00:00:00
    ```

- `NOW()` ou `CURRENT_TIMESTAMP` → retorna a data **com hora exata** do momento da execução.  
  - Exemplo:
    ```sql
    SELECT NOW();               -- 2025-09-09 14:35:42.123456
    SELECT CURRENT_TIMESTAMP;   -- 2025-09-09 14:35:42.123456
    ```

📌 Por isso, ao filtrar períodos por **dias inteiros** em colunas `timestamp`, sempre prefira o padrão:

```sql
WHERE dat_creation >= CURRENT_DATE - 15
  AND dat_creation < CURRENT_DATE
```

Isso garante que o intervalo inclua **todos os dias completos** até ontem, sem cortar registros.

---

## 🔑 Resumindo
- `::DATE` → simplifica, mas converte (menos performático).  
- `BETWEEN ... AND CURRENT_DATE - 1` → corta o último dia.  
- `>= CURRENT_DATE - 15 AND < CURRENT_DATE` → **forma recomendada**.  
- `CURRENT_DATE` = data atual, começa em `00:00:00`.  
- `NOW()` / `CURRENT_TIMESTAMP` = data + hora do momento atual.  

---

📌 **Nota interna**: Sempre que possível, prefira trabalhar com intervalos abertos em `timestamp` (`>= inicio` e `< fim`) em vez de `BETWEEN`, pois isso garante que o último dia seja considerado por completo e melhora a performance com índices.
