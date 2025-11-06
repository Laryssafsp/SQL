# Analyzing Students' Mental Health

Does going to university in a different country affect your mental health? A Japanese international university surveyed its students in 2018 and published a study the following year that was approved by several ethical and regulatory boards.

The study found that international students have a higher risk of mental health difficulties than the general population, and that social connectedness (belonging to a social group) and acculturative stress (stress associated with joining a new culture) are predictive of depression.


Explore the `students` data using PostgreSQL to find out if you would come to a similar conclusion for international students and see if the length of stay is a contributing factor.

Here is a data description of the columns you may find **helpful.**

| Field Name    | Description                                      |
| ------------- | ------------------------------------------------ |
| `inter_dom`     | Types of students (international or domestic)   |
| `japanese_cate` | Japanese language proficiency                    |
| `english_cate`  | English language proficiency                     |
| `academic`      | Current academic level (undergraduate or graduate) |
| `age`           | Current age of student                           |
| `stay`          | Current length of stay in years                  |
| `todep`         | Total score of depression (PHQ-9 test)           |
| `tosc`          | Total score of social connectedness (SCS test)   |
| `toas`          | Total score of acculturative stress (ASISS test) |

---

1. Contar estudantes internacionais = 201
`SELECT
    COUNT(*) AS count_int
FROM students
WHERE inter_dom = 'Inter';`

2. Médias (PHQ, SCS, AS) apenas para estudantes internacionais = 
`SELECT
    ROUND(AVG(todep), 2) AS average_phq,
    ROUND(AVG(tosc), 2) AS average_scs,
    ROUND(AVG(toas), 2) AS average_as
FROM students
WHERE inter_dom = 'Inter';
`

3. Contar e calcular médias por tempo de permanência (stay)
`SELECT
    stay,
    COUNT(*) AS count_int,
    ROUND(AVG(todep), 2) AS average_phq,
    ROUND(AVG(tosc), 2) AS average_scs,
    ROUND(AVG(toas), 2) AS average_as
FROM students
WHERE inter_dom = 'international'
GROUP BY stay;
`
4. Comparar internacionais vs. domésticos

Isso permite verificar se realmente há diferenças significativas entre os dois grupos.
Exemplos do que perguntar
- Estudantes internacionais têm mais depressão que domésticos?
- Têm mais estresse aculturativo? (esperado)
- Têm menos conexão social?
  
`SELECT
    inter_dom,
    ROUND(AVG(todep), 2) AS avg_phq,
    ROUND(AVG(tosc), 2) AS avg_scs,
    ROUND(AVG(toas), 2) AS avg_as
FROM students
GROUP BY inter_dom;
`
5. Ver correlação entre tempo de permanência e saúde mental
`
SELECT
    stay,
    ROUND(AVG(todep), 2) AS avg_phq,
    ROUND(AVG(toas), 2) AS avg_as,
    ROUND(AVG(tosc), 2) AS avg_scs
FROM students
WHERE inter_dom = 'Inter'
GROUP BY stay
ORDER BY stay;
`
6. Ver diferenças por nível acadêmico (grad vs undergrad)
`
SELECT
    academic,
    ROUND(AVG(todep), 2) AS avg_phq,
    ROUND(AVG(toas), 2) AS avg_as,
    ROUND(AVG(tosc), 2) AS avg_scs
FROM students
GROUP BY academic;
`
7. Investigar relação entre idade e saúde mental
`
SELECT
    age,
    ROUND(AVG(todep), 2) AS avg_phq
FROM students
GROUP BY age
ORDER BY age;
`
8. Procurar outliers

Estudantes com:
- PHQ extremamente alto → risco de depressão severa
- ASISS muito alto → estresse intenso
- SCS muito baixo → isolamento
`
SELECT *
FROM students
WHERE todep > 20 OR toas > 100 OR tosc < 30;
`
8. Contagem por idade
   

9. Faixas etárias recomendadas - Apenas uma análise, pois a quantidade de alunos vai diminuindo conforme mais velho. Deixando discrepante a análise

Essas são categorias comuns em análises de estudantes universitários:

| Faixa     | Idade                              |
| --------- | ---------------------------------- |
| **17–20** | Jovens iniciando a graduação       |
| **21–23** | Ciclo central da graduação         |
| **24–26** | Final da graduação / início da pós |
| **27–29** | Pós-graduação                      |
| **30+**   | Estudantes mais velhos             |

`
SELECT
    CASE
        WHEN age BETWEEN 18 AND 20 THEN '17-20'
        WHEN age BETWEEN 21 AND 23 THEN '21-23'
        WHEN age BETWEEN 24 AND 26 THEN '24-26'
        WHEN age BETWEEN 27 AND 29 THEN '27-29'
        ELSE '30+'
    END AS age_group,
    COUNT(*) AS count_students,
    ROUND(AVG(todep), 2) AS avg_phq,
    ROUND(AVG(tosc), 2) AS avg_scs,
    ROUND(AVG(toas), 2) AS avg_as
FROM students
GROUP BY age_group
ORDER BY age_group;
`
✅ Solução: reduzir o número de faixas e usar grupos mais amplos

| Faixa     | Idade                                  | Justificativa               |
| --------- | -------------------------------------- | --------------------------- |
| **18–20** | Início típico da graduação             | Grupo grande e homogêneo    |
| **21–24** | Meio da graduação / início da pós      | Maior amostra               |
| **25+**   | Pós-graduação e estudantes mais velhos | Evita grupos muito pequenos |

`
SELECT
    CASE
        WHEN age BETWEEN 18 AND 20 THEN '18-20'
        WHEN age BETWEEN 21 AND 24 THEN '21-24'
        ELSE '25+'
    END AS age_group,
    COUNT(*) AS count_students,
    ROUND(AVG(todep), 2) AS avg_phq,
    ROUND(AVG(tosc), 2) AS avg_scs,
    ROUND(AVG(toas), 2) AS avg_as
FROM students
WHERE inter_dom = 'international'
GROUP BY age_group
ORDER BY age_group;
`
# DF Final

`sql
SELECT
    stay,
    COUNT(*) AS count_int,
    ROUND(AVG(todep), 2) AS average_phq,
    ROUND(AVG(tosc), 2) AS average_scs,
    ROUND(AVG(toas), 2) AS average_as
FROM students
WHERE inter_dom = 'Inter'
GROUP BY stay
ORDER BY stay DESC;

`
