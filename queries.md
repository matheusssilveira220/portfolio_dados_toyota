```sql
CREATE VIEW toyota_grafico_controle AS
-- CTE onde agrupo os dados mensais e calculo a soma dos valores da coluna.
WITH 
metrica AS (
  SELECT 
    date(date_trunc('month', data)) AS dt,
    sum(round(alta::numeric, 2)) AS alta_total
  FROM toyota_stock
  GROUP BY 1
  ORDER BY 1
),

-- CTE onde cálculo a amplitude móvel
amplitudes AS (
  SELECT 
    dt,
    alta_total,
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
