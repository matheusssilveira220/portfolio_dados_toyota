# Query
Nessa query busco criar os dados para montar o gráfico de controle referente aos valores de Alta e Baixa do período, encontrando o percentual de alteração entre eles.

Foi realizado uma query dessa para cana ano até 2014. Após o periodo de 2014 foi realizada uma query contendo o periodo de 1980 até 2013, de forma apenas onde apresenta os dados. 

Os dados que usaremos para nossa análise serão principalmente os do período de 2014 a 2024


```sql
CREATE VIEW toyota_grafico_2024_controle AS
-- CTE onde agrupo os dados mensais e calculo a soma dos valores da coluna.
WITH 
metrica AS (
  SELECT 
    date(date_trunc('month', data)) AS dt,
    sum(round(alta::numeric, 2)) AS alta_total,
    sum(round(baixa::numeric, 2)) AS baixa_total
  FROM toyota_stock
  WHERE EXTRACT (YEAR FROM data) = 2024
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
