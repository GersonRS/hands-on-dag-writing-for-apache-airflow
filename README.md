# Hands on | Melhores Práticas na Criação de DAGs com Apache Airflow


## Agenda

- **DAG structure** best practices
- **Airflow project organization** best practices
- How to write **dynamic** and **scalable** pipeline
- **Testing and development** best practices
- **Common mistakes** that slow down your DAGs
- Demo
- Resources

## Overview

Este repositório contém o código para a demonstração do encontro Hands on | Melhores Práticas na Criação de DAGs com Apache Airflow

## Conteúdo

Este repositório contém:

- Um DAG mostrando práticas ruins do Airflow [`dags/bad_examples/bad_dag.py`](dags/bad_examples/bad_dag.py).
- Um DAG mostrando como o DAG ruim pode ser melhorado para usar boas práticas do Airflow [`dags/good_examples/good_dag.py`](dags/good_examples/good_dag.py).
- Um script para gerar dinamicamente arquivos DAG a partir de uma configuração JSON usando os arquivos em [`include/dynamic_dag_generation`](include/dynamic_dag_generation).
- Um exemplo de [fluxo de trabalho CI/CD](.github/workflows/deploy_to_astro.yaml) usando GitHub Actions para testar e implantar DAGs no [Astro](https://www.astronomer.io/try-astro).

- Um pequeno pipeline de dados funcional ingerindo informações sobre vendas, feedback do cliente e dados do cliente de uma empresa de brinquedos para cães. Para saber como executar esse pipeline, consulte a seção abaixo.

## Como executar este repositório

1. Clone o repositório.
2. Certifique-se de ter o [Astro CLI](https://docs.astronomer.io/astro/cli/install-cli) instalado e que o [Docker](https://www.docker.com/products/docker-desktop) esteja em execução.
3. Copie o arquivo `.env.example` para um novo arquivo chamado `.env` e preencha os detalhes da conexão do Snowflake e da AWS.
4. Crie um novo bucket na sua conta da AWS com uma pasta chamada `ingest`.
5. Copie as 3 pastas contendo um CSV cada de [`include/data_generation/data/ingest`](include/data_generation/data/ingest) para a pasta `ingest` no seu bucket. Você pode gerar mais dados executando o script [`include/data_generation/generate_sample_data.py`](include/data_generation/generate_sample_data.py).
6. Crie um novo banco de dados na sua conta do Snowflake chamado `HAPPYWOOFSDWH` com um esquema chamado `HAPPYWOOFSDEV`.
7. Na sua conta Snowflake crie 3 novos estágios para poder executar o DAG `load_to_snowflake` usando o SQL abaixo.

```sql
CREATE STAGE sales_reports_stage
URL = 's3://<your-bucket-name>/load/sales_reports/'
CREDENTIALS = (AWS_KEY_ID = '<your aws key id>' AWS_SECRET_KEY = '<your aws secret>')
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);

CREATE STAGE customer_feedback_stage
URL = 's3://<your-bucket-name>/load/customer_feedback/'
CREDENTIALS = (AWS_KEY_ID = '<your aws key id>' AWS_SECRET_KEY = '<your aws secret>')
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);

CREATE STAGE customer_data_stage
URL = 's3://<your-bucket-name>/load/customer_data/'
CREDENTIALS = (AWS_KEY_ID = '<your aws key id>' AWS_SECRET_KEY = '<your aws secret>')
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);
```

8. Na sua conta Snowflake, crie uma nova tabela chamada `TESTER_DOGS_TABLE` e preencha-a com os dados de [`include/data_generation/data/dog_profiles.csv](include/data_generation/data/dog_profiles.csv).
9. Execute `astro dev start` para iniciar o servidor web e o agendador do Airflow.
10. Navegue até `localhost:8080` no seu navegador para veja a IU do Airflow.
11. Retome todos os DAGs. A primeira execução dos DAGs `ingest` iniciará automaticamente e acionará DAGs downstream via [Datasets](https://docs.astronomer.io/learn/airflow-datasets).

## Usando `dag.test()`

Antes de executar `dag.test()` em DAGs que importam módulos da pasta `include`, você precisa adicionar a pasta `include` à variável de ambiente `PYTHONPATH`. Você pode fazer isso executando o seguinte comando no terminal:

```bash
export PYTHONPATH="<absolute-path-to-your-astro-project>:$PYTHONPATH"
```