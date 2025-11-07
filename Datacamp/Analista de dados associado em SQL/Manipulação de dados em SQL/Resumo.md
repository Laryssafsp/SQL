## **Seus aprendizados recentes**
Você aprendeu sobre as aplicações práticas de subconsultas dentro da cláusula SELECT de consultas SQL. Subconsultas em SELECT são ferramentas poderosas para incorporar valores agregados em um conjunto de dados detalhado, permitindo cálculos e comparações complexas diretamente dentro da sua consulta. Aqui estão os pontos principais que você cobriu:

- Subconsultas em SELECT para Valores Agregados: Você descobriu como usar subconsultas em SELECT para retornar um único valor agregado, como o número total de partidas jogadas em todas as temporadas ou a média geral de gols marcados em uma partida. Esta técnica é essencial quando você precisa incluir dados agregados em uma consulta não agrupada.
- Cálculos Usando Subconsultas: Você explorou como subconsultas podem realizar cálculos sobre dados dentro do seu banco de dados. Por exemplo, calcular quanto uma pontuação individual se desvia de uma média.
- Sintaxe e Posicionamento: A sintaxe correta para incluir uma subconsulta em uma instrução SELECT foi discutida, enfatizando que a subconsulta deve retornar um único valor para evitar erros. A importância de posicionar filtros corretamente tanto na consulta principal quanto na subconsulta para garantir resultados precisos também foi destacada.
- Exemplo Prático: Você praticou a construção de uma consulta para calcular a média de gols por partida em cada liga de um país para a temporada 2013/2014, comparando-a com a média geral. A solução demonstrou como selecionar e arredondar os gols totais da liga e a média geral de gols totais usando subconsultas:

```sql
SELECT 
    l.name AS league,
    ROUND(AVG(m.home_goal + m.away_goal),2) AS avg_goals,
    (SELECT ROUND(AVG(home_goal + away_goal),2) 
     FROM match
     WHERE season = '2013/2014') AS overall_avg
FROM league AS l
LEFT JOIN match AS m
ON l.country_id = m.country_id
WHERE m.season = '2013/2014'
GROUP BY l.name;
```
