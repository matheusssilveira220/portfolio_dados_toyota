# Query
Aqui encontrará todas as queries que importei ao Power BI para criar a visualização do Dashboard.

## Sumário
[1 - Gráfico de controle](#Gráfico-de-controle)

[2 - Performance Anual e Mensal](#Performance-Anual-e-Mensal)

[3 - Máximos e Mínimos Anuais](#Máximos-e-Mínimos-Anuais)

[4 - Volatilidade Mensal](#Volatilidade-Mensal)

[5 - Intervalo Baseado em Percentis](#Intervalo-Baseado-em-Percentis)

## Gráfico de Controle
Nessa query busco montar o gráfico de controle referente aos valores de Alta e Baixa do período de 2020 a 2024, encontrando o percentual de alteração entre eles.

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
## Máximos e Mínimos Anuais
Query voltara a entender como os anos se compartaram referente ao fechamento, buscando a diferença entre o máximo e o mínimo.

```sql
SELECT 
    EXTRACT(YEAR FROM data) AS ano, 
    MAX(fechamento) AS max_fechamento,
    MIN(fechamento) AS min_fechamento,
    ROUND(((MAX(fechamento) - MIN(fechamento)) / MAX(fechamento) * 100)::numeric, 2) AS dif_total
FROM toyota_stock
GROUP BY ano
ORDER BY ano;
```
## Volatilidade Mensal
Query onde busco a volatilidade baseado no mês anterior, nessa query notamos a baixa volatilidade nos fechamentos, onde a maior difereça de um mês para o outro foi de 0.074 (7,4%)

```sql
WITH dados AS (
    SELECT 
        data,
        fechamento,
        LAG(fechamento) OVER (PARTITION BY DATE_TRUNC('month', data) ORDER BY data) AS lag_fechamento,
        DATE_TRUNC('month', data) AS mes
    FROM toyota_stock
)
SELECT 
    mes,
    STDDEV((fechamento - lag_fechamento) / lag_fechamento) AS volatilidade
FROM dados
WHERE lag_fechamento IS NOT NULL
GROUP BY mes
ORDER BY volatilidade DESC;
```
## Intervalo Baseado em Percentis
Query buscando a diferença percentual anual, e os separando em quartis para analisarmos. Servindo como um apoio as queries anteriores, essa nos mostra que as ações da Toyota tem uma variação baixissima de periodo para periodo.

```sql
WITH dados_anuais AS (
    SELECT 
        EXTRACT(YEAR FROM data) AS ano,
        MAX(fechamento) AS max_fechamento,
        MIN(fechamento) AS min_fechamento,
        ROUND(
            ((MAX(fechamento::numeric) - MIN(fechamento::numeric)) / MAX(fechamento::numeric)) * 100, 
            2
        ) AS dif_total
    FROM toyota_stock
    GROUP BY ano
),
percentis AS (
    SELECT 
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY dif_total) AS p25,
        PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY dif_total) AS p50,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY dif_total) AS p75,
        PERCENTILE_CONT(1.0) WITHIN GROUP (ORDER BY dif_total) AS p100
    FROM dados_anuais
),
intervalos AS (
    SELECT 
        CASE
            WHEN dif_total <= p25 THEN '0% a 25%'
            WHEN dif_total <= p50 THEN '25% a 50%'
            WHEN dif_total <= p75 THEN '50% a 75%'
            ELSE '75% a 100%'
        END AS faixa_diferenca,
        COUNT(*) AS frequencia
    FROM dados_anuais, percentis
    GROUP BY faixa_diferenca
    ORDER BY faixa_diferenca
)
SELECT * FROM intervalos;
```
