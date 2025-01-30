# [Ver outros projetos](https://github.com/matheusssilveira220/portfolio_dados)
---
# Toyota Prices

## Introdução
O objetivo deste projeto é criar e analisar um Gráfico de Controle para identificar padrões de desempenho e apresentá-los em um dashboard interativo.

## Resumo

Nesse projeto fiz a análise historica do valor das ações da Toyota de 1980 a 2022.

Utilizei Python para extrair os dados da WEB utilizando API, fiz o tratamento dos dados e importei ao PostgreSQL.

No PostgreSQL, foram realizadas as consultas utilizando SQL, a projeção inicial e a Query especifica para gerar a visualização desejada.

A visualização foi gerada utilizando o PowerBI.

O fluxo do trabalho pode ser acompanhado no arquivo [fluxo_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/fluxo_toyota_0.2.pdf) e o escopo do projeto estará no arquivo [escopo_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/escopo_toyota.pdf).

Os objetivos, juntando o fluxo + escopo, estão no arquivo [objetivos_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/objetivos_toyota.pdf).

## Etapas

### Python

Fiz a coleta dos dados utilizando o API do Kaggle, após isso realizei a análise exploratoria e o tratamento nos dados. Com isso chegava a etapa de conectar ao banco de dados, fiz isso utilizando a biblioteca **sqlalchemy**, biblioteca voltada a integração de Python - SQL. A própria plataforma da **Supabase** nos oferece os codigos necessários para realizar essa importação dos dados ao banco de dados.

### SQL

Utilizei a plataforma da **Supabase** para hospedagem do meu banco de dados, com a ferramenta **DBeaver** como ferramenta visual para operarmos as queries futuramente. O SGBD que utilizei é o **PostgreSQL**. Dessa forma tenho todo suporte necessário para começar minhas análises dentro do SQL.
