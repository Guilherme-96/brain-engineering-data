# 🔷 Azure — Engenharia de Dados

> Plataforma cloud da Microsoft. Dominante em empresas que usam o ecossistema Microsoft.

**Relacionado:** [[AWS]] | [[Databricks]] | [[Docker]] | [[Terraform]] | [[PowerBI]] | [[PostgreSQL]] | [[Spark_PySpark]] | [[Escolha_de_Stack]]

---

## Serviços de Dados Azure — Visão Geral

| Serviço Azure | Equivalente AWS | Função |
|--------------|-----------------|--------|
| Azure Data Lake Storage Gen2 (ADLS) | S3 | Armazenamento de objetos / Data Lake |
| Azure Synapse Analytics | Redshift + Glue | Data Warehouse + ETL integrado |
| Azure Data Factory (ADF) | AWS Glue + Step Functions | Orquestração de pipelines |
| Azure Databricks | EMR + Databricks | Spark gerenciado |
| Azure SQL Database | RDS | SQL Server gerenciado |
| Azure Cosmos DB | DynamoDB | NoSQL multi-modelo |
| Azure Stream Analytics | Kinesis Analytics | Streaming em tempo real |
| Azure Event Hubs | Kinesis Data Streams | Ingestão de eventos em massa |
| Azure Service Bus | SQS + SNS | Mensageria |
| Azure Functions | Lambda | Serverless |
| Azure Container Apps | ECS Fargate | Containers serverless |
| Power BI | QuickSight | BI e dashboards |
| Azure Purview | AWS Glue Catalog | Governança de dados |
| Azure Key Vault | Secrets Manager | Segredos e chaves |

---

## ADLS Gen2 — Azure Data Lake Storage
```python
from azure.storage.blob import BlobServiceClient
from azure.identity import DefaultAzureCredential
import pandas as pd
from io import StringIO, BytesIO

# Autenticação (via Service Principal ou Managed Identity)
credential = DefaultAzureCredential()
account_url = "https://minha_conta.dfs.core.windows.net"
client = BlobServiceClient(account_url=account_url, credential=credential)

container = client.get_container_client("data-lake")

# Upload
with open("dados.parquet", "rb") as f:
    container.upload_blob("bronze/vendas/2024/dados.parquet", f, overwrite=True)

# Download para Pandas
blob = container.download_blob("silver/clientes/clientes.parquet")
df = pd.read_parquet(BytesIO(blob.readall()))

# Listar arquivos
for blob in container.list_blobs(name_starts_with="bronze/vendas/"):
    print(blob.name, blob.size)

# Com Pandas direto (usando abfs URI)
# pip install adlfs
df = pd.read_parquet(
    "abfs://data-lake@minha_conta.dfs.core.windows.net/silver/vendas/*.parquet",
    storage_options={
        "account_name": "minha_conta",
        "tenant_id": "xxx",
        "client_id": "xxx",
        "client_secret": "xxx"
    }
)
```

---

## Azure Data Factory — Orquestração Visual
```json
// Pipeline ADF em JSON (exportado)
{
  "name": "pipeline_vendas",
  "properties": {
    "activities": [
      {
        "name": "CopiarDadosSQLServer",
        "type": "Copy",
        "inputs": [{"referenceName": "SqlServerDataset"}],
        "outputs": [{"referenceName": "ADLSParquetDataset"}],
        "typeProperties": {
          "source": {"type": "SqlSource", "sqlReaderQuery": "SELECT * FROM vendas"},
          "sink": {"type": "ParquetSink"}
        }
      },
      {
        "name": "TransformarComDatabricks",
        "dependsOn": [{"activity": "CopiarDadosSQLServer"}],
        "type": "DatabricksNotebook",
        "typeProperties": {
          "notebookPath": "/pipelines/transformar_vendas"
        }
      }
    ]
  }
}
```

**ADF vs Airflow:**
- ADF: visual, sem código, integração nativa com Azure, ótimo para equipes de dados não-Python
- [[Airflow]]: código Python, mais flexível, melhor para engenheiros

---

## Azure Synapse Analytics
```sql
-- Synapse SQL Pool (Data Warehouse)
-- Distribuição de dados para performance

CREATE TABLE fato_vendas (
    id_venda    BIGINT,
    data        DATE,
    valor       DECIMAL(15,2),
    cliente_id  INT
)
WITH (
    DISTRIBUTION = HASH(cliente_id),   -- distribui pelos nodes
    CLUSTERED COLUMNSTORE INDEX        -- compressão colunar
);

-- External Table — consultar ADLS diretamente!
CREATE EXTERNAL TABLE vendas_externas (
    id      INT,
    valor   DECIMAL(15,2),
    data    DATE
)
WITH (
    LOCATION = 'bronze/vendas/**',
    DATA_SOURCE = meu_datalake,
    FILE_FORMAT = ParquetFormat
);

-- Consulta sobre parquet no ADLS sem mover dados!
SELECT DATE_TRUNC('month', data), SUM(valor)
FROM vendas_externas
GROUP BY 1;
```

---

## Azure Databricks — Integração Nativa
```python
# Em notebook Databricks na Azure — acesso ADLS sem configuração!
# (via Unity Catalog ou montagem automática)

# Ler do ADLS Gen2
df = spark.read.parquet("abfss://data-lake@conta.dfs.core.windows.net/bronze/vendas/")

# Escrever como Delta
df.write.format("delta").mode("overwrite") \
    .save("abfss://data-lake@conta.dfs.core.windows.net/silver/vendas/")

# Criar tabela gerenciada no Unity Catalog
spark.sql("""
    CREATE TABLE IF NOT EXISTS producao.vendas.fato_vendas
    USING DELTA
    LOCATION 'abfss://data-lake@conta.dfs.core.windows.net/gold/fato_vendas/'
""")

# Conectar ao Azure SQL Database
jdbcUrl = "jdbc:sqlserver://servidor.database.windows.net:1433;database=meudb"
df_sql = spark.read.format("jdbc") \
    .option("url", jdbcUrl) \
    .option("dbtable", "dbo.clientes") \
    .option("authentication", "ActiveDirectoryMSI") \
    .load()
```

---

## Infraestrutura com Terraform
```hcl
provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "dados" {
  name     = "rg-pipeline-dados"
  location = "Brazil South"
}

# Storage Account para Data Lake
resource "azurerm_storage_account" "datalake" {
  name                     = "datalakepipeline"
  resource_group_name      = azurerm_resource_group.dados.name
  location                 = azurerm_resource_group.dados.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  is_hns_enabled           = true   # ADLS Gen2
}

# Container (equivalente ao bucket no S3)
resource "azurerm_storage_container" "bronze" {
  name                  = "bronze"
  storage_account_name  = azurerm_storage_account.datalake.name
  container_access_type = "private"
}

# Azure SQL Database
resource "azurerm_mssql_server" "sql" {
  name                         = "sql-pipeline-dados"
  resource_group_name          = azurerm_resource_group.dados.name
  location                     = azurerm_resource_group.dados.location
  version                      = "12.0"
  administrator_login          = "adminuser"
  administrator_login_password = var.db_password
}

resource "azurerm_mssql_database" "db" {
  name      = "pipeline-db"
  server_id = azurerm_mssql_server.sql.id
  sku_name  = "S0"   # 10 DTUs — dev
}
```

---

## ✅ Quando escolher Azure vs AWS

**Escolha Azure quando:**
- Empresa já usa Microsoft 365, Teams, SharePoint
- Precisa de integração nativa com Power BI (via Synapse)
- SQL Server é o banco principal
- Azure DevOps é o CI/CD da empresa
- Databricks é o motor de dados (parceria histórica)

**Escolha AWS quando:**
- Maior ecossistema de serviços de dados
- Empresa cloud-native sem legado Microsoft
- Precisa de Redshift, EMR, Kinesis
