# Slowly Changing Dimensions (SCD)

## O que são Slowly Changing Dimensions?

**Slowly Changing Dimensions (SCDs)** são técnicas utilizadas em projetos de **Business Intelligence (BI)** e **Data Warehousing** para gerenciar e acompanhar mudanças nos dados dimensionais ao longo do tempo.

Esses dados dimensionais representam informações descritivas — como nomes de clientes, endereços, produtos, entre outros — que podem sofrer alterações com o tempo. As SCDs garantem que essas mudanças sejam tratadas de forma adequada, permitindo análises precisas tanto do estado atual quanto do histórico dos dados.

As SCDs são fundamentais para manter a integridade histórica dos dados, especialmente em sistemas analíticos. Por exemplo, uma empresa pode querer analisar as vendas considerando o endereço do cliente na data da compra, e não o endereço atual.

Existem **sete tipos principais de SCDs**: Tipo 0, 1, 2, 3, 4, 5 e 6. Cada um lida de forma distinta com as atualizações nos dados dimensionais.

---

## Tipos de Slowly Changing Dimensions

### 🔒 SCD Tipo 0 – Retenção de Dados (Sem Mudanças)

- Os dados não são atualizados.
- Qualquer tentativa de alteração é ignorada.
- Usado quando o dado deve permanecer fixo (ex.: CPF, CNPJ).

### 🔁 SCD Tipo 1 – Sobrescrita Simples

- Os dados antigos são sobrescritos pelos novos.
- **Não há histórico** de alterações.
- Simples de implementar, mas a perda de histórico pode ser um problema.

### 📜 SCD Tipo 2 – Histórico Completo

- Cada alteração gera uma **nova linha** na tabela de dimensões.
- Permite manter um **histórico completo** das mudanças.
- Comum usar:
  - Colunas de **data de início e fim**.
  - Coluna de **versão** (ex.: V1, V2, etc.).
  - Coluna de **status** (ativo/inativo), geralmente combinada com as opções acima.

### 🔄 SCD Tipo 3 – Histórico Limitado

- Mantém apenas o **valor atual** e o **valor anterior** em colunas separadas.
- Indicado quando apenas **uma mudança** precisa ser registrada.

### 🗃️ SCD Tipo 4 – Tabela Histórica Separada

- A tabela principal armazena apenas o **dado atual**.
- Uma **tabela separada** armazena todas as mudanças históricas.
- Boa opção para isolar o histórico e manter a performance da dimensão principal.

### 🧩 SCD Tipo 5 – Tipo 1 + Tipo 4

- Combina os métodos do **Tipo 1** e **Tipo 4**.
- A dimensão principal é atualizada com o novo dado (**sobrescrita**).
- A tabela histórica separada registra todas as versões anteriores.

### 🧬 SCD Tipo 6 – Dimensão Híbrida (Tipo 1 + 2 + 3)

- Combina os métodos dos tipos **1, 2 e 3**.
- Permite sobrescrever valores, criar registros históricos e armazenar mudanças limitadas — tudo na mesma estrutura.
- É a abordagem mais completa e flexível, porém mais complexa de implementar.

---

## Exemplos de Slowly Changing Dimensions (SCD)

### 🔒 SCD Tipo 0 – Retenção de Dados (Sem Mudanças)

**Situação:** Um cliente altera o nome, mas a empresa decide não atualizar esse dado.

| ID Cliente | Nome       | CPF           |
|------------|------------|---------------|
| 1          | João Silva | 123.456.789-00 |

> **Nova tentativa de alteração:** Nome atualizado para "João S. Oliveira"  
> **Resultado:** Nome continua sendo "João Silva" – alteração ignorada.

---

### 🔁 SCD Tipo 1 – Sobrescrita Simples

**Situação:** Cliente muda de cidade, e a empresa atualiza o dado diretamente.

**Antes:**

| ID Cliente | Nome       | Cidade     |
|------------|------------|------------|
| 1          | Ana Souza  | São Paulo  |

**Após atualização:**

| ID Cliente | Nome       | Cidade     |
|------------|------------|------------|
| 1          | Ana Souza  | Campinas   |

> **Histórico perdido:** Não é possível saber que Ana Souza morava em São Paulo.

---

### 📜 SCD Tipo 2 – Histórico Completo

**Situação:** Cliente muda de cidade, e o histórico é mantido com múltiplas linhas.

| ID Dimensão | ID Cliente | Nome       | Cidade     | Início Vigência | Fim Vigência | Status  |
|-------------|------------|------------|------------|------------------|--------------|---------|
| 101         | 1          | Ana Souza  | São Paulo  | 2022-01-01       | 2023-03-15   | Inativo |
| 102         | 1          | Ana Souza  | Campinas   | 2023-03-16       | NULL         | Ativo   |

> **Consulta histórica:** Podemos saber onde o cliente morava em qualquer data.

---

### 🔄 SCD Tipo 3 – Histórico Limitado

**Situação:** Mantemos o valor atual e o valor anterior de uma cidade.

| ID Cliente | Nome       | Cidade Atual | Cidade Anterior |
|------------|------------|---------------|------------------|
| 1          | Ana Souza  | Campinas      | São Paulo        |

> **Limitação:** Se houver mais de uma mudança, só se mantém o último histórico.

---

### 🗃️ SCD Tipo 4 – Tabela Histórica Separada

**Dimensão Principal:**

| ID Cliente | Nome       | Cidade     |
|------------|------------|------------|
| 1          | Ana Souza  | Campinas   |

**Tabela Histórica:**

| ID Histórico | ID Cliente | Nome       | Cidade     | Data da Mudança |
|--------------|------------|------------|------------|------------------|
| 1            | 1          | Ana Souza  | São Paulo  | 2022-01-01       |
| 2            | 1          | Ana Souza  | Campinas   | 2023-03-16       |

> **Boa prática:** Histórico não interfere na performance da tabela principal.

---

### 🧩 SCD Tipo 5 – Tipo 1 + Tipo 4

**Dimensão Principal (sobrescrita):**

| ID Cliente | Nome       | Cidade     |
|------------|------------|------------|
| 1          | Ana Souza  | Campinas   |

**Tabela Histórica (versões anteriores):**

| ID Histórico | ID Cliente | Cidade     | Vigência Início | Vigência Fim |
|--------------|------------|------------|------------------|--------------|
| 1            | 1          | São Paulo  | 2022-01-01       | 2023-03-15   |
| 2            | 1          | Campinas   | 2023-03-16       | NULL         |

> **Flexível:** Combina performance com histórico detalhado.

---

### 🧬 SCD Tipo 6 – Dimensão Híbrida (1 + 2 + 3)

**Situação:** Um cliente muda de cidade e queremos:
- Manter o histórico (Tipo 2),
- Atualizar o dado atual (Tipo 1),
- Armazenar o valor anterior (Tipo 3).

| ID Dimensão | ID Cliente | Nome       | Cidade Atual | Cidade Anterior | Início Vigência | Fim Vigência | Status  |
|-------------|------------|------------|----------------|------------------|------------------|--------------|---------|
| 102         | 1          | Ana Souza  | Campinas       | São Paulo        | 2023-03-16       | NULL         | Ativo   |

> **Nota:** Essa estrutura precisa de lógica extra no ETL, mas é altamente informativa.

---

