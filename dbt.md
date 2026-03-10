# 🔧 dbt — Data Build Tool

> Ferramenta de transformação de dados SQL com versionamento, testes e documentação. O "git para SQL".

**Relacionado:** [[PostgreSQL]] | [[Databricks]] | [[AWS]] | [[Azure]] | [[Modelagem_de_Dados]] | [[Governanca_de_Dados]] | [[Airflow]] | [[DuckDB_Multiengine]]

---

## O que é dbt?

dbt faz apenas o **T** do ETL → **EL**T

```
Você fornece:  SQL SELECT
dbt faz:       CREATE TABLE/VIEW + testes + documentação + lineage
```

**Sem dbt:**
```sql
-- SQL espalhado em arquivos sem padrão
-- Sem testes
-- Sem documentação
-- Difícil de entender dependências
```

**Com dbt:**
```sql
-- Models organizados em camadas
-- Testes automáticos
-- Docs gerados automaticamente
-- Grafo de dependências visual
```

---

## Instalação
```bash
pip install dbt-postgres       # para PostgreSQL
pip install dbt-bigquery       # para BigQuery
pip install dbt-snowflake      # para Snowflake
pip install dbt-databricks     # para Databricks
pip install dbt-duckdb         # para DuckDB (local)

dbt --version
```

---

## Estrutura do Projeto
```
meu_projeto_dbt/
├── dbt_project.yml          # configuração principal
├── profiles.yml             # conexões (não commitar!)
├── models/
│   ├── staging/             # camada bronze → prata (renomear, tipar)
│   │   ├── _sources.yml     # declarar fontes externas
│   │   ├── stg_pedidos.sql
│   │   └── stg_clientes.sql
│   ├── intermediate/        # lógica de negócio intermediária
│   │   └── int_pedidos_com_clientes.sql
│   └── marts/               # camada gold — consumo final
│       ├── fct_vendas.sql
│       └── dim_clientes.sql
├── tests/                   # testes customizados em SQL
├── macros/                  # funções SQL reutilizáveis
├── seeds/                   # CSVs pequenos (tabelas de referência)
├── snapshots/               # SCD Tipo 2 automático
└── analyses/                # SQL exploratório (não materializado)
```

---

## Models — SQL com Superpoderes
```sql
-- models/staging/stg_pedidos.sql
-- Referência a fontes com {{ source() }}
SELECT
    id_pedido,
    cliente_id,
    CAST(data_pedido AS DATE)  AS data_pedido,
    UPPER(TRIM(status))         AS status,
    valor::NUMERIC(15,2)        AS valor
FROM {{ source('raw', 'pedidos_brutos') }}
WHERE data_pedido IS NOT NULL


-- models/marts/fct_vendas.sql
-- Referência a outros models com {{ ref() }}
-- dbt resolve automaticamente a ordem de execução!
SELECT
    p.id_pedido,
    p.data_pedido,
    c.nome_cliente,
    c.cidade,
    p.valor,
    p.status
FROM {{ ref('stg_pedidos') }} p
LEFT JOIN {{ ref('stg_clientes') }} c ON c.id_cliente = p.cliente_id
```

---

## Materializations — Como dbt Cria os Objetos
```yaml
# dbt_project.yml
models:
  meu_projeto:
    staging:
      +materialized: view          # view — sempre atualizado, sem custo de storage
    intermediate:
      +materialized: ephemeral     # CTE — não cria objeto no banco
    marts:
      +materialized: table         # table — snapshot físico
      fct_vendas:
        +materialized: incremental # incremental — só processa dados novos!
```

```sql
-- Model incremental (ideal para tabelas grandes)
-- models/marts/fct_vendas.sql
{{ config(materialized='incremental', unique_key='id_pedido') }}

SELECT ...
FROM {{ ref('stg_pedidos') }}

{% if is_incremental() %}
    WHERE data_pedido > (SELECT MAX(data_pedido) FROM {{ this }})
{% endif %}
```

---

## Testes Automáticos
```yaml
# models/staging/_schema.yml
version: 2

models:
  - name: stg_pedidos
    description: "Pedidos limpos e tipados"
    columns:
      - name: id_pedido
        description: "Identificador único do pedido"
        tests:
          - not_null
          - unique
      
      - name: status
        tests:
          - not_null
          - accepted_values:
              values: ['APROVADO', 'PENDENTE', 'CANCELADO']
      
      - name: cliente_id
        tests:
          - not_null
          - relationships:
              to: ref('stg_clientes')
              field: id_cliente
      
      - name: valor
        tests:
          - not_null
          - dbt_utils.accepted_range:
              min_value: 0
              inclusive: false
```

---

## Snapshots — SCD Tipo 2 Automático
```sql
-- snapshots/snap_clientes.sql
{% snapshot snap_clientes %}
{{ config(
    target_schema='snapshots',
    unique_key='id_cliente',
    strategy='timestamp',
    updated_at='updated_at'
) }}

SELECT * FROM {{ source('raw', 'clientes') }}

{% endsnapshot %}
-- dbt snapshot → cria automaticamente colunas dbt_valid_from, dbt_valid_to
```

---

## Macros — SQL Reutilizável
```sql
-- macros/centavos_para_reais.sql
{% macro centavos_para_reais(coluna) %}
    ({{ coluna }} / 100.0)::NUMERIC(15,2)
{% endmacro %}

-- Usar no model:
SELECT
    id_pedido,
    {{ centavos_para_reais('valor_centavos') }} AS valor_reais
FROM ...
```

---

## Comandos
```bash
dbt run              # executar todos os models
dbt run --select stg_pedidos    # model específico
dbt run --select staging.*      # todos do diretório staging
dbt run --select +fct_vendas    # model + todos os dependentes (upstream)

dbt test             # rodar todos os testes
dbt test --select stg_pedidos

dbt docs generate    # gerar documentação
dbt docs serve       # servir docs em http://localhost:8080

dbt snapshot         # executar snapshots

dbt seed             # carregar arquivos CSV como tabelas

dbt debug            # testar conexão com o banco
dbt compile          # compilar SQL sem executar
```

---

## Integração com Airflow
```python
from airflow.operators.bash import BashOperator

dbt_run = BashOperator(
    task_id="dbt_run",
    bash_command="cd /opt/dbt && dbt run --profiles-dir /opt/dbt",
    env={"DBT_TARGET": "prod"}
)

dbt_test = BashOperator(
    task_id="dbt_test",
    bash_command="cd /opt/dbt && dbt test --profiles-dir /opt/dbt"
)

extrair >> dbt_run >> dbt_test >> notificar
```

---

## ✅ Quando usar dbt / ❌ Quando não usar

✅ USE dbt quando:
- Transformações são principalmente em SQL
- Precisa de testes automáticos nos dados
- Time quer documentação automática
- Destino é um Data Warehouse (Redshift, BigQuery, Snowflake, PostgreSQL)

❌ NÃO USE dbt quando:
- Transformações exigem Python puro (use [[Pandas]] ou [[Spark_PySpark]])
- É um pipeline de ingestão (dbt não faz Extract nem Load)
- Dados são muito pequenos e simples (um SELECT já basta)
