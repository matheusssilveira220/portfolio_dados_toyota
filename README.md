### [Ver outros projetos](https://github.com/matheusssilveira220/portfolio_dados)
---
![Badge em Desenvolvimento](http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=GREEN&style=for-the-badge)
# Toyota Prices

## Apresentação
O objetivo deste projeto é criar e analisar um Gráfico de Controle para identificar padrões de desempenho e apresentá-los em um dashboard interativo.

Link do Kaggle: [Toyota Motors Stock Data (1980-2024)](https://www.kaggle.com/datasets/mhassansaboor/toyota-motors-stock-data-2980-2024)

## Sumário
[1 - Como rodar o projeto](#Como-rodar-o-projeto)

[2 - Resumo](#Resumo)

[3 - Etapas](#Etapas)

[3.1 - Python](#Python)

[3.2 - SQL](#SQL)

[3.3 - PowerBI](#PowerBI)

## Como rodar o projeto

### Pré-requisitos

Antes de iniciar, certifique-se de ter instalado em sua máquina:

- Python 3.8+

- PostgreSQL

- Power BI

- Jupyter Notebook

- Git

### Clonar o repositório
```
git clone https://github.com/matheusssilveira220/portfolio_dados_toyota.git
cd portfolio_dados_toyota
```
### Criar e Ativar um Ambiente Virtual (Opcional, mas Recomendado)
```
pip install -r requirements.txt
```
### Configurar o Banco de Dados

Certifique-se de que o PostgreSQL está rodando na sua máquina ou em um servidor remoto.
Crie um banco de dados chamado toyota_db (ou outro de sua escolha).
Configure as credenciais no arquivo .env:
```
user=seu_usuario
password=sua_senha
host=localhost
port=5432
dbname=toyota_db
```
### Rode as migrações para criar a tabela necessária

```sql
CREATE TABLE toyota_stock (
    data DATE,
    fechamento_ajustado FLOAT,
    fechamento FLOAT,
    alta FLOAT,
    baixa FLOAT,
    abertura FLOAT,
    volume INT
);
```

### Rodar o Código
```
python toyota.py
```
# Resumo

Nesse projeto fiz a análise historica do valor das ações da Toyota de 1980 a 2022.

Utilizei Python para extrair os dados da WEB utilizando API, fiz o tratamento dos dados e importei ao PostgreSQL.

No PostgreSQL, foram realizadas as consultas utilizando SQL, a projeção inicial e a Query especifica para gerar a visualização desejada.

A visualização foi gerada utilizando o PowerBI.

O fluxo do trabalho pode ser acompanhado no arquivo [fluxo_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/fluxo_toyota_0.2.pdf) e o escopo do projeto estará no arquivo [escopo_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/escopo_toyota.pdf).

Os objetivos, juntando o fluxo + escopo, estão no arquivo [objetivos_toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/objetivos_toyota.pdf).

## Etapas

### Python

Fiz a coleta dos dados utilizando o API do Kaggle, após isso realizei a análise exploratoria e o tratamento nos dados. Com isso chegava a etapa de conectar ao banco de dados, fiz isso utilizando a biblioteca **sqlalchemy**, biblioteca voltada a integração de Python - SQL. A própria plataforma da **Supabase** nos oferece os codigos necessários para realizar essa importação dos dados ao banco de dados.

No arquivo [toyota](https://github.com/matheusssilveira220/portfolio_dados_toyota/blob/main/toyota.ipynb) vocês encontram os notebooks criados utilizando o **JupyterLab**.

### SQL 

Utilizei a plataforma da **Supabase** para hospedagem do meu banco de dados, com a ferramenta **DBeaver** como ferramenta visual para operarmos as queries futuramente. O SGBD que utilizei é o **PostgreSQL**. Dessa forma tenho todo suporte necessário para começar minhas análises dentro do SQL.

### PowerBI

O [Dashboard PowerBI](https://app.powerbi.com/view?r=eyJrIjoiNWY1ZjAwZDktMDA3MC00OTUwLThiZDgtNjc0NGMyMzY1NzQyIiwidCI6IjljMTdjMzNhLWJhMGItNDM0MS04ZjI5LTZjYWVkYjAxN2NhMyJ9&pageName=a246d69471eb9602b248) pode ser visto nesse link destacado.
