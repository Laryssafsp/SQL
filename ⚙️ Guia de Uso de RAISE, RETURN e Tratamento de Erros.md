# ⚙️ Guia de Uso de `RAISE`, `RETURN` e Tratamento de Erros em PL/pgSQL

> **Objetivo:** Definir boas práticas para utilização de `RAISE`, `RETURN` e controle de erros em procedures PL/pgSQL.
> **Aplicável a:** PostgreSQL e Amazon Redshift.

---

## 📘 1. Tipos de `RAISE` e Quando Utilizar

| Tipo              | Interrompe Execução | Finalidade                                         | Exemplo                                                    |
| ----------------- | ------------------- | -------------------------------------------------- | ---------------------------------------------------------- |
| `RAISE NOTICE`    | ❌ Não               | Informar progresso, etapas e status da execução    | `RAISE NOTICE 'Iniciando processo de carga';`              |
| `RAISE INFO`      | ❌ Não               | Log detalhado para depuração                       | `RAISE INFO 'Tempo de execução: %.2f segundos', v_tempo;`  |
| `RAISE WARNING`   | ❌ Não               | Alertar sobre situações incomuns, mas não críticas | `RAISE WARNING 'Tabela vazia, prosseguindo sem inserção';` |
| `RAISE EXCEPTION` | ✅ Sim               | Interromper execução por erro grave                | `RAISE EXCEPTION 'Erro ao atualizar tabela: %', SQLERRM;`  |

### Boas práticas:

* Use `NOTICE` para logs normais.
* Use `WARNING` para eventos não fatais.
* Use `EXCEPTION` apenas quando a execução não pode continuar.

---

## 🔁 2. Uso do `RETURN` e Variações

| Comando        | Tipo de Função                      | Descrição                                   | Exemplo                               |
| -------------- | ----------------------------------- | ------------------------------------------- | ------------------------------------- |
| `RETURN`       | Procedure ou função simples         | Encerra imediatamente a execução            | `IF v_total = 0 THEN RETURN; END IF;` |
| `RETURN NEXT`  | Função que retorna múltiplas linhas | Retorna uma linha parcial                   | `RETURN NEXT v_registro;`             |
| `RETURN QUERY` | Função que retorna *setof*          | Retorna o resultado de uma consulta inteira | `RETURN QUERY SELECT * FROM tabela;`  |
| `RETURN NULL`  | Função escalar                      | Encerra sem retorno de valor                | `IF v_erro THEN RETURN NULL; END IF;` |

> Em procedures (`CREATE PROCEDURE`), apenas `RETURN` é permitido; funções (`CREATE FUNCTION`) podem usar as demais variações.

---

## 🧩 3. Estrutura Recomendada de Procedure com Tratamento de Erros

```sql
CREATE OR REPLACE PROCEDURE sp_exemplo_raises()
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INT;
BEGIN
    RAISE NOTICE 'Início da execução.';

    SELECT COUNT(*) INTO v_total FROM tabela_origem;

    IF v_total = 0 THEN
        RAISE NOTICE 'Nenhum registro encontrado. Encerrando procedure.';
        RETURN;
    END IF;

    BEGIN
        INSERT INTO tabela_destino SELECT * FROM tabela_origem;
    EXCEPTION
        WHEN unique_violation THEN
            RAISE WARNING 'Registro duplicado identificado.';
        WHEN OTHERS THEN
            RAISE EXCEPTION 'Erro inesperado: %', SQLERRM;
    END;

    RAISE NOTICE 'Execução concluída com sucesso.';
END;
$$;
```

---

## ⚙️ 4. Quando Usar Cada Tipo de Encerramento (`RETURN`)

| Situação                     | Ação Recomendada            | Exemplo                                                       |
| ---------------------------- | --------------------------- | ------------------------------------------------------------- |
| Nenhum dado novo a processar | `RAISE NOTICE` + `RETURN`   | `RAISE NOTICE 'Sem novos registros'; RETURN;`                 |
| Erro crítico                 | `RAISE EXCEPTION`           | `RAISE EXCEPTION 'Falha ao atualizar dados';`                 |
| Evento incomum mas não fatal | `RAISE WARNING` + continuar | `RAISE WARNING 'Campo nulo detectado, continuando processo';` |
| Finalização normal           | `RAISE NOTICE`              | `RAISE NOTICE 'Processo finalizado com sucesso';`             |

---

## 🧱 5. Exemplo Completo com Bloco de Exceção

```sql
CREATE OR REPLACE PROCEDURE sp_exemplo_completo()
LANGUAGE plpgsql
AS $$
DECLARE
    v_contador INT;
BEGIN
    RAISE NOTICE 'Iniciando processamento.';

    SELECT COUNT(*) INTO v_contador FROM tabela_origem;

    IF v_contador = 0 THEN
        RAISE NOTICE 'Nenhum dado encontrado. Encerrando.';
        RETURN;
    END IF;

    BEGIN
        -- Ação principal
        INSERT INTO tabela_destino SELECT * FROM tabela_origem;
    EXCEPTION
        WHEN OTHERS THEN
            RAISE WARNING 'Erro capturado: %', SQLERRM;
            RAISE EXCEPTION 'Execução interrompida.';
    END;

    RAISE NOTICE 'Processamento concluído.';
END;
$$;
```

---

## 🧠 6. Padrão de Mensagens de Log

```
[NÍVEL] [AÇÃO] [OBJETO] [DETALHE OPCIONAL]
```

**Exemplos:**

```
NOTICE Iniciando carga de tabela principal.
WARNING Nenhum registro novo identificado.
EXCEPTION Falha ao inserir dados: duplicate key value.
```

> Mantenha as mensagens curtas, contextuais e padronizadas.

---

## ✅ 7. Checklist de Boas Práticas

* [x] Adicionar `RAISE NOTICE` no início e ao final da execução.
* [x] Validar volume de registros antes de operações críticas.
* [x] Utilizar `RETURN` para encerramento limpo.
* [x] Incluir bloco `EXCEPTION` para tratamento de erros.
* [x] Usar mensagens de log consistentes e significativas.
* [x] Evitar `RAISE EXCEPTION` genérico sem contexto.
* [x] Garantir que as exceções não ocultem erros reais (sempre exibir `SQLERRM`).

---

## ⏱️ 8. Exemplo com Medição de Tempo (Opcional)

```sql
DECLARE
    v_inicio TIMESTAMP := clock_timestamp();
    v_fim TIMESTAMP;
    v_execucao INTERVAL;
BEGIN
    RAISE NOTICE 'Processamento iniciado.';
    
    -- Lógica principal

    v_fim := clock_timestamp();
    v_execucao := v_fim - v_inicio;
    RAISE INFO 'Tempo total de execução: %', v_execucao;
END;
```

---

> **Resumo Final:**
>
> * Use `RAISE` para registrar o comportamento da procedure.
> * Sempre trate erros com blocos `EXCEPTION`.
> * Encerre execuções de forma limpa com `RETURN`.
> * Mantenha mensagens claras, objetivas e padronizadas.


---

## Exemplo: Tratando ausência de registros como situação normal

Em procedures PL/pgSQL, se a ausência de registros **não deve ser considerada um erro**, é recomendável usar `RAISE NOTICE` para logar a situação e `RETURN;` para encerrar a execução de forma limpa.

```sql
IF v_qtde_registros = 0 THEN
    RAISE NOTICE 'Nenhum registro novo para inserir. Encerrando procedure.';
    RETURN;  -- encerra a execução sem erro
END IF;
```

### 💡 Observações:

- `RAISE NOTICE` **não interrompe** a execução, apenas registra uma mensagem informativa.
- `RETURN;` encerra a procedure imediatamente quando não há dados a processar.
- Este padrão é útil para **procedures de carga de dados**, onde não ter novos registros é uma situação esperada.
- Evita lançar `RAISE EXCEPTION` em cenários que **não são erros reais**, mantendo logs claros e sem falhas desnecessárias.

### ✅ Vantagens:

- Procedimento encerra de forma segura.
- Logs ficam claros para auditoria.
- Evita erros falsos que poderiam interromper pipelines ou jobs automatizados.
```


