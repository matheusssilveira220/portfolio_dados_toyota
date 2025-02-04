# Query
## Gráfico de Controle
Nessa query busco montar o gráfico de controle referente aos valores de Alta e Baixa do período de 2024 a 2020, encontrando o percentual de alteração entre eles.

A intenção do gráfico é encontrar pontos de sazonlidade e alterações bruscas nos valores apresentados.


```sql
CREATE VIEW toyota_grafico_controle_controle AS
-- CTE onde agrupo os dados mensais e calculo a soma dos valores da coluna.
WITH 
metrica AS (
  SELECT 
    date(date_trunc('month', data)) AS dt,
    sum(round(alta::numeric, 2)) AS alta_total,
    sum(round(baixa::numeric, 2)) AS baixa_total
  FROM toyota_stock
  WHERE EXTRACT (YEAR FROM data) >= 2020 
  GROUP BY 1
  ORDER BY 1
),

-- CTE onde cálculo a amplitude móvel
amplitudes AS (
  SELECT 
    dt,
    alta_total,
    baixa_total,
    abs(alta_total - coalesce(lag(alta_total) OVER (ORDER BY dt), alta_total)) AS amplitude --Faço o uso de coalesce para evitar o uso de outra cte
  FROM metrica
),

-- CTE cálculo as médias para os limites de controle
medias AS (
  SELECT
    round(avg(alta_total), 2) AS media_geral,
    round(avg(amplitude), 2) AS media_amplitude
  FROM amplitudes
  WHERE amplitude IS NOT NULL
),

-- CTE para definir os limites de controle
limites AS (
  SELECT
    3.27 * media_amplitude AS las, --Valor retirado de gráficos de controle XMR
    media_geral + (2.66 * media_amplitude) AS lns, --Valor retirado de gráficos de controle XMR
    media_geral - (2.66 * media_amplitude) AS lni --Valor retirado de gráficos de controle XMR
  FROM medias
)

-- Select final com tudo junto
SELECT
  a.dt,
  a.alta_total,
  a.baixa_total,
  ROUND(((a.alta_total - a.baixa_total)/a.baixa_total) * 100, 2) AS diferenca,
  m.media_geral,
  a.amplitude,
  l.las AS limite_amplitude_superior,
  l.lns AS limite_natural_superior,
  l.lni AS limite_natural_inferior
FROM amplitudes a
CROSS JOIN limites l
CROSS JOIN medias m
ORDER BY a.dt;
```
## Performance Anual e Mensal
A intensão dessa query é encontrar anos e meses com positividade e negatividade, considerando o fechamento, ou seja caso fechamento > fechamento anterior = 1. Dessa forma conseguimos encontrar os anos e meses com maior taxa de positividade de um periodo ao outro. A query é a mesma nos dois casos, apenas alterar MONTH por YEAR e mes por ano.

Com isso encontramos também uma tendência de queda ou alta nas ações, baseado nos dados historicos.

```sql
-- Prepara os dados brutos com informações da data anterior
WITH dados AS (
    SELECT 
        data,
        fechamento,
        LAG(fechamento) OVER (ORDER BY data) AS lag_fechamento,
        EXTRACT(month FROM data) AS mes -->Altere month para year, e tera o resultado anual, o Alias é interessante mudar para ano
    FROM toyota_stock
),
-- Classifica cada data como alta/baixa.
contagem_dias AS (
    SELECT 
        mes, --> Alterar para ano
        data,
        fechamento,
        CASE WHEN fechamento > lag_fechamento THEN 1 ELSE 0 END AS alta,
        CASE WHEN fechamento < lag_fechamento THEN 1 ELSE 0 END AS baixa
    FROM dados
)
-- Select final com tudo junto.
SELECT 
    mes, --> Alterar para ano
    SUM(alta) AS total_dias_alta,
    SUM(baixa) AS total_dias_baixa,
    CASE WHEN SUM(alta) > SUM(baixa) THEN 'Positivo' ELSE 'Negativo' END AS positivo_negativo
FROM contagem_dias
GROUP BY mes --> Alterar para ano
ORDER BY mes; --> Alterar para ano
```
