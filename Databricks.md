# 🧱 Databricks

> Plataforma unificada de dados e IA baseada em Apache Spark. Criada pelos fundadores do Spark.

**Relacionado:** [[Spark_PySpark]] | [[AWS]] | [[Azure]] | [[dbt]] | [[DuckDB_Multiengine]] | [[Airflow]] | [[Modelagem_de_Dados]] | [[Escolha_de_Stack]]

---

## O que é Databricks?

Databricks é uma **plataforma gerenciada** que une:
- Apache [[Spark_PySpark]] (processamento distribuído)
- Delta Lake (formato de tabela ACID sobre S3/ADLS)
- MLflow (versionamento e serving de modelos ML)
- Notebooks colaborativos (como Jupyter, mas compartilhado)
- Unity Catalog (governança centralizada)
- SQL Warehouse (consultas analíticas sem Spark)

```
Sem Databricks:
Você instala Spark → configura cluster → gerencia workers → cuida de Delta Lake → ...

Com Databricks:
Você abre o notebook e começa a escrever código.
```

---

## Delta Lake — O Coração do Databricks

```python
from delta.tables import DeltaTable
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Escrever como Delta (ACID, schema evolution, time travel)
df.write.format("delta").mode("overwrite").save("s3://bucket/delta/vendas/")

# Ler
df = spark.read.format("delta").load("s3://bucket/delta/vendas/")

# Upsert (MERGE) — operação poderosa
delta_tabela = DeltaTable.forPath(spark, "s3://bucket/delta/clientes/")

delta_tabela.alias("existente").merge(
    df_novos.alias("novos"),
    "existente.id = novos.id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()

# Time Travel — consultar dados do passado!
df_ontem = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-15") \
    .load("s3://bucket/delta/vendas/")

df_versao_3 = spark.read.format("delta") \
    .option("versionAsOf", 3) \
    .load("s3://bucket/delta/vendas/")

# Ver histórico de versões
DeltaTable.forPath(spark, "s3://bucket/delta/vendas/").history().show()
```

---

## Arquitetura Medalhão (Medallion Architecture)

```
Bronze  → dados brutos, imutáveis, exatamente como chegaram
Silver  → limpos, validados, sem duplicatas
Gold    → agregados, prontos para negócio

s3://bucket/
├── bronze/
│   ├── vendas/year=2024/month=01/vendas_raw.json
│   └── clientes/clientes_raw.csv
├── silver/
│   ├── vendas/               ← Delta Table
│   └── clientes/             ← Delta Table
└── gold/
    ├── fct_vendas/           ← Delta Table particionada
    └── dim_clientes/         ← Delta Table
```

```python
# Pipeline Bronze → Silver → Gold

# Bronze: ingestão raw
df_bronze = spark.read.json("s3://bucket/raw/vendas/*.json")
df_bronze.write.format("delta").mode("append") \
    .partitionBy("ano", "mes") \
    .save("s3://bucket/bronze/vendas/")

# Silver: limpeza
df_silver = df_bronze \
    .filter(F.col("valor").isNotNull()) \
    .withColumn("data", F.to_date("data_str", "yyyy-MM-dd")) \
    .dropDuplicates(["id_pedido"])

df_silver.write.format("delta").mode("overwrite") \
    .save("s3://bucket/silver/vendas/")

# Gold: agregação
df_gold = df_silver \
    .groupBy(F.month("data").alias("mes"), "categoria") \
    .agg(F.sum("valor").alias("total_vendas"))

df_gold.write.format("delta").mode("overwrite") \
    .save("s3://bucket/gold/fct_vendas_mensal/")
```

---

## Unity Catalog — Governança Centralizada
```sql
-- Três níveis: catalog.schema.table
-- Exemplo: producao.vendas.fct_pedidos

-- Criar estrutura
CREATE CATALOG producao;
CREATE SCHEMA producao.vendas;

-- Acessar
SELECT * FROM producao.vendas.fct_pedidos;

-- Controle de acesso granular
GRANT SELECT ON TABLE producao.vendas.fct_pedidos TO grupo_analistas;
GRANT SELECT, MODIFY ON SCHEMA producao.vendas TO engenheiros_dados;

-- Column masking (mascarar dados sensíveis por role)
CREATE ROW FILTER IF NOT EXISTS filtro_regiao
ON producao.vendas.fct_pedidos
USING COLUMNS (regiao)
RETURN is_member('grupo_' || regiao);
```

---

## Databricks SQL — Warehouse Analítico
```sql
-- SQL sem precisar de cluster Spark
-- Conecta ao Power BI, Tableau, Metabase

-- Criar tabela gerenciada
CREATE TABLE IF NOT EXISTS vendas.fct_pedidos
USING DELTA
PARTITIONED BY (ano, mes)
AS SELECT * FROM bronze.pedidos_raw;

-- Performance: Z-Ordering (otimização de layout para filtros comuns)
OPTIMIZE vendas.fct_pedidos ZORDER BY (cliente_id, data_pedido);

-- Auto Optimize
ALTER TABLE vendas.fct_pedidos SET TBLPROPERTIES (
    delta.autoOptimize.optimizeWrite = true,
    delta.autoOptimize.autoCompact = true
);
```

---

## Integração com dbt
```bash
pip install dbt-databricks

# profiles.yml
databricks_profile:
  target: prod
  outputs:
    prod:
      type: databricks
      host: adb-xxx.azuredatabricks.net
      http_path: /sql/1.0/warehouses/xxx
      token: "{{ env_var('DATABRICKS_TOKEN') }}"
      catalog: producao
      schema: vendas
```

---

## ✅ Quando usar Databricks / ❌ Quando não usar

✅ USE Databricks quando:
- Dados em TB ou PB
- Time já usa Spark
- Precisa de Delta Lake com ACID + Time Travel
- ML + dados na mesma plataforma
- Empresa usa Azure (Databricks é nativo na Azure)

❌ NÃO USE Databricks quando:
- Dados são pequenos (< 10 GB) — use [[DuckDB_Multiengine]] ou Pandas
- Budget é restrito — é caro! Use Spark OSS + EMR
- Time não tem expertise em Spark
- Só precisa de SQL analítico simples — use Redshift ou BigQuery
