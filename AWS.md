# ☁️ AWS — Amazon Web Services

> Principal plataforma de cloud computing. Dominar os serviços de dados da AWS é essencial.

**Relacionado:** [[Docker]] | [[Terraform]] | [[Airflow]] | [[Spark_PySpark]] | [[PostgreSQL]] | [[Python]]

---

## Serviços de Dados — Visão Geral

| Serviço | Função | Quando Usar |
|---------|--------|-------------|
| **S3** | Armazenamento de objetos / Data Lake | Sempre — base de tudo |
| **RDS** | Banco relacional gerenciado | PostgreSQL/MySQL em produção |
| **Redshift** | Data Warehouse analítico | Analytics em petabytes |
| **Glue** | ETL serverless + Catálogo | Transformações sem servidor |
| **Athena** | SQL sobre S3 | Consultas ad-hoc baratas |
| **EMR** | Spark/Hadoop gerenciado | Big Data pesado |
| **Kinesis** | Streaming em tempo real | Ingestão contínua |
| **Lambda** | Funções serverless | Triggers, pequenos processos |
| **EC2** | Servidores virtuais | Controle total do ambiente |
| **ECS/EKS** | Containers | Orquestração Docker/K8s |
| **IAM** | Gerenciamento de acesso | Permissões e segurança |
| **Secrets Manager** | Armazenar credenciais | Senhas, tokens, keys |
| **CloudWatch** | Monitoramento e logs | Alertas e observabilidade |
| **SNS/SQS** | Mensageria | Desacoplar serviços |

---

## S3 — Simple Storage Service
```python
import boto3
from pathlib import Path

# Cliente S3
s3 = boto3.client(
    "s3",
    region_name="us-east-1",
    # Credenciais via variável de ambiente ou IAM Role (recomendado)
    # aws_access_key_id=..., aws_secret_access_key=...
)

# Upload
s3.upload_file("local.csv", "meu-bucket", "raw/dados.csv")

# Com metadados
s3.upload_file(
    "local.csv", "meu-bucket", "raw/dados.csv",
    ExtraArgs={"ContentType": "text/csv", "ServerSideEncryption": "AES256"}
)

# Download
s3.download_file("meu-bucket", "raw/dados.csv", "local/dados.csv")

# Listar objetos
paginator = s3.get_paginator("list_objects_v2")
for page in paginator.paginate(Bucket="meu-bucket", Prefix="raw/2024/"):
    for obj in page.get("Contents", []):
        print(obj["Key"], obj["Size"])

# Ler direto com Pandas (sem baixar)
import pandas as pd
df = pd.read_csv("s3://meu-bucket/raw/dados.csv")
df.to_parquet("s3://meu-bucket/processed/dados.parquet")

# Copiar entre buckets
s3.copy_object(
    CopySource={"Bucket": "origem", "Key": "arquivo.csv"},
    Bucket="destino",
    Key="arquivo.csv"
)

# Deletar
s3.delete_object(Bucket="meu-bucket", Key="raw/arquivo.csv")

# URL pré-assinada (acesso temporário)
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": "meu-bucket", "Key": "relatorio.pdf"},
    ExpiresIn=3600  # 1 hora
)
```

---

## Organização de Buckets (Data Lake)
```
meu-data-lake/
├── raw/                    # dados brutos, imutáveis
│   ├── vendas/2024/01/
│   │   └── vendas_20240101.csv
│   └── clientes/
│       └── clientes.json
├── processed/              # dados limpos e transformados
│   ├── vendas/
│   │   └── year=2024/month=01/  # particionado
│   │       └── data.parquet
│   └── clientes/
└── curated/                # dados prontos para consumo
    └── dm_vendas/
        └── data.parquet
```

---

## IAM — Credenciais Seguras
```python
# NÃO faça isso (hardcoded)
boto3.client("s3", aws_access_key_id="AKIA...", aws_secret_access_key="...")

# Faça isso — variáveis de ambiente
import os
# export AWS_ACCESS_KEY_ID=AKIA...
# export AWS_SECRET_ACCESS_KEY=...
# export AWS_DEFAULT_REGION=us-east-1
boto3.client("s3")  # boto3 pega automaticamente

# Melhor ainda — use IAM Role (sem credenciais no código)
# EC2, Lambda, ECS → associar role com permissões S3
boto3.client("s3")  # usa a role associada automaticamente

# Secrets Manager — para senhas de banco, tokens de API
secretsmanager = boto3.client("secretsmanager")
secret = secretsmanager.get_secret_value(SecretId="prod/minha-app/postgres")
import json
credenciais = json.loads(secret["SecretString"])
```

---

## Lambda — Funções Serverless
```python
# handler.py — função Lambda
import json
import boto3
import pandas as pd
from io import StringIO

def lambda_handler(event, context):
    """Trigger: S3 Event Notification quando arquivo chega."""
    
    bucket = event["Records"][0]["s3"]["bucket"]["name"]
    key = event["Records"][0]["s3"]["object"]["key"]
    
    s3 = boto3.client("s3")
    obj = s3.get_object(Bucket=bucket, Key=key)
    df = pd.read_csv(obj["Body"])
    
    # Processar
    df_processado = transformar(df)
    
    # Salvar no bucket de processados
    buffer = StringIO()
    df_processado.to_csv(buffer, index=False)
    s3.put_object(
        Bucket="processed-bucket",
        Key=key.replace("raw/", "processed/"),
        Body=buffer.getvalue()
    )
    
    return {"statusCode": 200, "body": json.dumps("Processado!")}
```

---

## CLI AWS
```bash
# Instalar
pip install awscli

# Configurar
aws configure
# AWS Access Key ID: ...
# AWS Secret Access Key: ...
# Default region name: us-east-1
# Default output format: json

# S3
aws s3 ls                                         # listar buckets
aws s3 ls s3://meu-bucket/raw/                   # listar objetos
aws s3 cp local.csv s3://meu-bucket/raw/          # upload
aws s3 cp s3://meu-bucket/raw/arquivo.csv .       # download
aws s3 sync ./dados/ s3://meu-bucket/dados/       # sincronizar pasta
aws s3 rm s3://meu-bucket/arquivo.csv             # deletar

# Verificar identidade
aws sts get-caller-identity
```
