# 🦆 DuckDB e Multi-Engine ETL

> DuckDB é um banco de dados analítico in-process. O SQLite para analytics. Surpreendentemente rápido.

**Relacionado:** [[Python]] | [[Pandas]] | [[Spark_PySpark]] | [[dbt]] | [[Databricks]] | [[AWS]] | [[Escolha_de_Stack]]

---

## O que é DuckDB?

```
SQLite     → banco embedded para OLTP (transacional)
DuckDB     → banco embedded para OLAP (analítico)

Sem servidor. Sem configuração. SQL moderno. Absurdamente rápido.
```

**DuckDB lê nativamente:**
- Parquet (local e S3)
- CSV e JSON
- Arrow / Pandas DataFrames
- PostgreSQL (via extensão)
- SQLite
- Excel (via extensão)

---

## Instalação e Uso Básico
```bash
pip install duckdb
```

```python
import duckdb

# Conexão em memória
con = duckdb.connect()

# Banco persistente
con = duckdb.connect("meu_banco.duckdb")

# SQL direto em arquivo Parquet — sem carregar na memória!
resultado = con.execute("""
    SELECT 
        categoria,
        SUM(valor) as total,
        COUNT(*) as qtd
    FROM 'dados/*.parquet'
    GROUP BY categoria
    ORDER BY total DESC
""").df()   # retorna Pandas DataFrame

# Ler CSV
con.execute("SELECT * FROM 'dados.csv' LIMIT 10").show()

# Ler S3 (com httpfs extension)
con.execute("INSTALL httpfs; LOAD httpfs;")
con.execute("""
    SET s3_region='us-east-1';
    SET s3_access_key_id='...';
    SET s3_secret_access_key='...';
""")
df = con.execute("SELECT * FROM 's3://bucket/dados/*.parquet'").df()
```

---

## DuckDB + Pandas — O Melhor dos Dois Mundos
```python
import duckdb
import pandas as pd

df = pd.read_csv("vendas_gigante.csv")  # 5GB de CSV...

# DuckDB opera DIRETAMENTE no DataFrame Pandas — sem copiar!
resultado = duckdb.execute("""
    SELECT 
        DATE_TRUNC('month', CAST(data AS DATE)) as mes,
        SUM(valor) as total,
        AVG(valor) as media,
        COUNT(*) as qtd
    FROM df
    WHERE status = 'APROVADO'
    GROUP BY 1
    ORDER BY 1
""").df()

# Muito mais rápido que pandas puro para queries analíticas!
# DuckDB usa execução vetorizada (SIMD)
```

---

## DuckDB vs Pandas vs Spark

| Critério | Pandas | DuckDB | Spark |
|----------|--------|--------|-------|
| Volume ideal | < RAM | < Disco | Qualquer |
| SQL | Limitado | SQL completo | SQL |
| Setup | pip install | pip install | Cluster |
| Velocidade analítica | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Joins grandes | Lento | Muito rápido | Distribuído |
| Window Functions | Parcial | Completo | Completo |
| Curva aprendizado | Baixa | Baixa (SQL) | Alta |

---

## Multi-Engine ETL — Usando a Ferramenta Certa em Cada Etapa

```python
# Estratégia: cada engine onde é mais forte

import duckdb
import pandas as pd
from pyspark.sql import SparkSession

# PASSO 1: Ingestão pesada (100GB de Parquet no S3)
# → Spark: distribuído, paralelismo nativo em S3
spark = SparkSession.builder.appName("ETL").getOrCreate()
df_spark = spark.read.parquet("s3://bucket/raw/**/*.parquet") \
    .filter("ano = 2024") \
    .groupBy("cliente_id") \
    .agg({"valor": "sum"})
df_spark.write.parquet("/tmp/agregado/")

# PASSO 2: Transformações analíticas complexas
# → DuckDB: SQL poderoso, window functions, JOINs pesados localmente
con = duckdb.connect()
df_resultado = con.execute("""
    WITH ranked AS (
        SELECT *,
               RANK() OVER (PARTITION BY categoria ORDER BY total DESC) as rank
        FROM '/tmp/agregado/*.parquet'
    )
    SELECT * FROM ranked WHERE rank <= 10
""").df()

# PASSO 3: Limpeza e transformações em Python puro
# → Pandas: manipulação flexível, apply, string processing
df_final = df_resultado \
    .assign(nome_upper=lambda df: df['nome'].str.upper()) \
    .dropna(subset=['email'])

# PASSO 4: Carga no destino
# → dbt ou SQLAlchemy para carregar no Data Warehouse
df_final.to_parquet("s3://bucket/gold/top_clientes.parquet")
```

---

## DuckDB com dbt
```bash
pip install dbt-duckdb

# profiles.yml
duckdb_profile:
  outputs:
    dev:
      type: duckdb
      path: dev.duckdb      # banco local para desenvolvimento
    prod:
      type: duckdb
      path: s3://bucket/prod.duckdb  # banco no S3
```

```sql
-- model dbt rodando em DuckDB
-- lê Parquet do S3 diretamente!
SELECT 
    DATE_TRUNC('month', data_pedido) AS mes,
    SUM(valor) AS total
FROM read_parquet('s3://bucket/bronze/vendas/*.parquet')
GROUP BY 1
```

---

## ✅ Quando usar DuckDB / ❌ Quando não usar

✅ USE DuckDB quando:
- Analytics em arquivos locais ou S3 (até ~100GB)
- Substituir Pandas em consultas analíticas SQL
- Desenvolvimento local (sem precisar de cluster)
- Backend de dbt para prototipar transformações
- ETL simples sem orquestração complexa

❌ NÃO USE DuckDB quando:
- Dados não cabem no disco local
- Precisa de concorrência alta de escrita (OLTP)
- Precisa de processamento verdadeiramente distribuído
- Team precisa de interface web (use PostgreSQL ou Databricks)
