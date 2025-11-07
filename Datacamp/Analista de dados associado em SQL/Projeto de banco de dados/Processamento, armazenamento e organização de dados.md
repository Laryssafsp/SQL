# Modelagem de Dados

## 1. Modelo de Dados Conceitual
- **Objetivo:** Reunir requisitos de negócios e representar o que é importante para a organização.  
- **Foco:** Entidades, atributos e relacionamentos.  
- **Características:**  
  - Independente de tecnologia ou sistemas de armazenamento.  
  - Visão abstrata do negócio.  
- **Saída típica:** Diagrama ER (Entidade-Relacionamento).

---

## 2. Modelo de Dados Lógico
- **Objetivo:** Transformar o modelo conceitual em um modelo formal, mas ainda independente de SGBD.  
- **Foco:** Estrutura de dados organizada, normalização, definição de tabelas e colunas.  
- **Características:**  
  - Mantém entidades, atributos e relacionamentos.  
  - Define chaves primárias, chaves estrangeiras e integridade referencial.  
- **Saída típica:** Esquema relacional lógico com tabelas, colunas, chaves primárias e estrangeiras.

---

## 3. Modelo de Dados Físico
- **Objetivo:** Mapear o modelo lógico para a tecnologia específica (SGBD).  
- **Foco:** Estrutura de arquivos de armazenamento, índices, tipos de dados específicos, particionamento e performance.  
- **Características:**  
  - Depende do SGBD e do hardware.  
  - Define tabelas, colunas, índices, restrições, triggers e estratégias de armazenamento.  
- **Saída típica:** Scripts SQL de criação de tabelas, índices, particionamento.

---

## Resumo da Evolução

| Nível        | Foco                                 | Saída Típica                                         |
|-------------|-------------------------------------|-----------------------------------------------------|
| Conceitual   | Requisitos de negócio               | Diagrama ER (entidades, atributos, relacionamentos)|
| Lógico       | Estrutura de dados, normalização     | Modelo relacional, tabelas, colunas, chaves        |
| Físico       | Implementação no SGBD               | Scripts SQL, arquivos, índices, particionamento    |
