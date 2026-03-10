# ⚡ Apache Spark / PySpark

> Framework de processamento distribuído para big data. Processa terabytes em minutos.

**Relacionado:** [[Python]] | [[Pandas]] | [[AWS]] | [[Docker]] | [[Airflow]] | [[Modelagem_de_Dados]]

---

## Conceitos Fundamentais
- **RDD:** Resilient Distributed Dataset — estrutura de dados distribuída e tolerante a falhas
- **DataFrame:** API tabular de alto nível (similar ao Pandas)
- **Partition:** divisão dos dados para processamento paralelo
- **Driver:** processo principal que coordena o trabalho
- **Executor:** processos que executam as tasks nas máquinas
- **Lazy Evaluation:** transformações só executam quando uma ação é chamada
- **DAG:** Directed Acyclic Graph — plano de execução interno do Spark

---

## Instalação Local
```bash
pip install pyspark
# Requer Java (JDK 8 ou 11)
sudo apt install default-jdk
```

---

## SparkSession
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MeuPipeline") \
    .master("local[*]")  \               # local com todos os cores
    .config("spark.executor.memory", "4g") \
    .config("spark.driver.memory", "2g") \
    .config("spark.sql.shuffle.partitions", "200") \
    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")   # reduzir logs verbosos
```

---

## Leitura de Dados
```python
# CSV
df = spark.read.csv("dados.csv", header=True, inferSchema=True)
df = spark.read \
    .option("header", True) \
    .option("sep", ";") \
    .option("encoding", "UTF-8") \
    .csv("dados/*.csv")           # lê múltiplos arquivos

# Parquet (recomendado para big data)
df = spark.read.parquet("s3://bucket/dados/")

# JSON
df = spark.read.json("dados.json")

# Delta Lake
df = spark.read.format("delta").load("s3://bucket/delta/tabela/")

# Banco de dados
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://localhost:5432/banco") \
    .option("dbtable", "vendas") \
    .option("user", "admin") \
    .option("password", "senha") \
    .load()
```

---

## Transformações (Lazy)
```python
from pyspark.sql import functions as F
from pyspark.sql.types import *
from pyspark.sql.window import Window

# Seleção e filtro
df.select("col1", "col2")
df.filter(F.col("ativo") == True)
df.filter("valor > 100 AND status = 'aprovado'")
df.where(F.col("data") >= "2024-01-01")

# Novas colunas
df.withColumn("valor_com_taxa", F.col("valor") * 1.1)
df.withColumn("data", F.to_date("data_str", "yyyy-MM-dd"))
df.withColumn("mes", F.month(F.col("data")))
df.withColumn("categoria", 
    F.when(F.col("valor") > 1000, "alto")
     .when(F.col("valor") > 100, "medio")
     .otherwise("baixo")
)

# Renomear e dropar
df.withColumnRenamed("old_name", "new_name")
df.drop("coluna_inutil")

# Distinct e deduplicação
df.distinct()
df.dropDuplicates(["id"])

# Ordenar
df.orderBy("valor", ascending=False)
df.orderBy(F.col("data").desc(), F.col("nome").asc())

# Limitar
df.limit(100)
```

---

## Agregações
```python
# groupBy
resultado = df \
    .groupBy("categoria", F.month("data").alias("mes")) \
    .agg(
        F.sum("valor").alias("total"),
        F.count("*").alias("qtd"),
        F.avg("valor").alias("media"),
        F.max("valor").alias("maximo"),
        F.min("valor").alias("minimo"),
        F.countDistinct("cliente_id").alias("clientes_unicos"),
        F.collect_list("produto").alias("produtos"),  # lista de valores
        F.stddev("valor").alias("desvio_padrao")
    ) \
    .orderBy("total", ascending=False)
```

---

## Joins
```python
# Inner join
df_resultado = df_pedidos.join(df_clientes, on="cliente_id", how="inner")

# Múltiplas chaves
df_a.join(df_b, on=["id", "data"], how="left")

# Chaves diferentes
df_a.join(df_b, df_a.user_id == df_b.id, how="left")

# Tipos: inner, left, right, full, cross, semi, anti
# semi: retorna linhas da esquerda que TÊM match (sem colunas da direita)
# anti: retorna linhas da esquerda que NÃO TÊM match
```

---

## Window Functions
```python
window_depto = Window.partitionBy("departamento").orderBy("salario")
window_tempo = Window.orderBy("data").rowsBetween(Window.unboundedPreceding, Window.currentRow)

df.withColumn("rank", F.rank().over(window_depto)) \
  .withColumn("media_depto", F.avg("salario").over(Window.partitionBy("departamento"))) \
  .withColumn("acumulado", F.sum("valor").over(window_tempo)) \
  .withColumn("lag_valor", F.lag("valor", 1).over(window_tempo))
```

---

## SQL no Spark
```python
df.createOrReplaceTempView("vendas")

resultado = spark.sql("""
    WITH mensais AS (
        SELECT 
            DATE_TRUNC('month', data) AS mes,
            categoria,
            SUM(valor) AS total
        FROM vendas
        WHERE ativo = true
        GROUP BY 1, 2
    )
    SELECT 
        mes,
        categoria,
        total,
        SUM(total) OVER (PARTITION BY categoria ORDER BY mes) AS acumulado
    FROM mensais
    ORDER BY mes, total DESC
""")
```

---

## Ações (Executam o plano)
```python
df.show(20)                  # exibe no console
df.show(20, truncate=False)  # sem truncar strings longas
df.count()                   # conta linhas
df.collect()                 # traz TODOS para o driver (cuidado!)
df.first()                   # primeira linha
df.take(5)                   # primeiras 5 linhas
df.printSchema()             # exibe schema
df.describe().show()         # estatísticas
df.toPandas()               # converte para Pandas (CUIDADO com memória!)
```

---

## Escrita
```python
df.write \
    .mode("overwrite") \           # overwrite, append, ignore, error
    .partitionBy("ano", "mes") \   # particiona os arquivos
    .parquet("s3://bucket/output/")

df.write.mode("append").json("saida/")

# JDBC
df.write \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://localhost/banco") \
    .option("dbtable", "vendas_spark") \
    .option("user", "admin") \
    .option("password", "senha") \
    .mode("overwrite") \
    .save()
```

---

## Performance
```python
# Cache — armazena em memória para reutilização
df.cache()
df.persist(StorageLevel.MEMORY_AND_DISK)

# Repartição
df.repartition(200)          # aumentar paralelismo
df.coalesce(1)               # reduzir para 1 arquivo de saída

# Broadcast join — para tabelas pequenas (evita shuffle)
from pyspark.sql.functions import broadcast
df_grande.join(broadcast(df_pequeno), on="id")

# Ver plano de execução
df.explain(True)

spark.stop()
```
