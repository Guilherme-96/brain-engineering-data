# ⚡ Event-Driven com Amazon SQS e Lambda

> Arquitetura orientada a eventos: sistemas que reagem a acontecimentos em vez de serem chamados diretamente.

**Relacionado:** [[AWS]] | [[Python]] | [[FastAPI]] | [[Airflow]] | [[Docker]] | [[Observabilidade_Pipelines]] | [[Kafka]]

---

## O que é Arquitetura Event-Driven?

```
Modelo tradicional (request-response):
Cliente → chama diretamente → Serviço → responde

Modelo event-driven:
Produtor → publica evento → Fila/Tópico → Consumidor reage
```

**Vantagens:**
- **Desacoplamento:** produtor não sabe quem vai consumir
- **Resiliência:** se o consumidor cair, mensagens ficam na fila
- **Escala:** múltiplos consumidores em paralelo
- **Retry automático:** SQS tenta reprocessar mensagens com falha

---

## Amazon SQS — Simple Queue Service

### Tipos de Fila
```
Standard Queue:
- Alta throughput (ilimitada)
- Entrega "at-least-once" (pode duplicar)
- Ordem não garantida
- Use para: processamento em massa, tolerante a duplicatas

FIFO Queue (.fifo):
- Ordem garantida (first-in, first-out)
- Exatamente uma entrega
- Throughput menor (300-3000 msgs/s)
- Use para: transações financeiras, ordem importa
```

### Operações Básicas com Python
```python
import boto3
import json
from typing import Any

sqs = boto3.client("sqs", region_name="us-east-1")
QUEUE_URL = "https://sqs.us-east-1.amazonaws.com/123456789/minha-fila"

# PUBLICAR mensagem
def publicar(payload: dict, delay_segundos: int = 0) -> str:
    response = sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(payload),
        DelaySeconds=delay_segundos,
        MessageAttributes={
            "tipo_evento": {
                "StringValue": payload.get("tipo", "desconhecido"),
                "DataType": "String"
            }
        }
    )
    return response["MessageId"]


# CONSUMIR mensagens
def consumir(max_mensagens: int = 10, timeout_visibilidade: int = 30) -> list:
    response = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=max_mensagens,
        WaitTimeSeconds=20,       # Long polling — mais eficiente!
        VisibilityTimeout=timeout_visibilidade,
        MessageAttributeNames=["All"]
    )
    return response.get("Messages", [])


# DELETAR após processar (confirmar sucesso)
def confirmar(receipt_handle: str):
    sqs.delete_message(
        QueueUrl=QUEUE_URL,
        ReceiptHandle=receipt_handle
    )


# Worker completo com tratamento de erro
def worker_pipeline():
    print("Worker iniciado. Aguardando mensagens...")
    
    while True:
        mensagens = consumir()
        
        for msg in mensagens:
            payload = json.loads(msg["Body"])
            receipt = msg["ReceiptHandle"]
            
            try:
                processar_evento(payload)
                confirmar(receipt)       # só confirma se processou com sucesso
                print(f"✅ Processado: {payload}")
                
            except Exception as e:
                print(f"❌ Erro ao processar: {e}")
                # NÃO confirmar → SQS vai reenviar após VisibilityTimeout
                # Após N falhas → vai para Dead Letter Queue (DLQ)
```

---

## AWS Lambda — Funções Serverless

### Trigger por SQS
```python
# lambda_function.py

import json
import boto3
import pandas as pd
from io import StringIO
from datetime import datetime

def lambda_handler(event, context):
    """
    Executado automaticamente quando chega mensagem na SQS.
    event["Records"] contém as mensagens do lote.
    """
    
    s3 = boto3.client("s3")
    processados = 0
    falhos = []
    
    for record in event["Records"]:
        msg_id = record["messageId"]
        
        try:
            payload = json.loads(record["body"])
            tipo = payload.get("tipo")
            
            if tipo == "arquivo_novo":
                processar_arquivo(s3, payload)
                
            elif tipo == "invalidar_cache":
                invalidar_cache(payload)
                
            elif tipo == "notificar":
                enviar_notificacao(payload)
                
            else:
                print(f"Tipo desconhecido: {tipo}")
            
            processados += 1
            
        except Exception as e:
            print(f"Erro na mensagem {msg_id}: {e}")
            falhos.append(msg_id)
    
    # Retornar falhos para SQS reprocessar (Partial Batch Response)
    return {
        "batchItemFailures": [
            {"itemIdentifier": msg_id} for msg_id in falhos
        ]
    }


def processar_arquivo(s3, payload: dict):
    bucket = payload["bucket"]
    key = payload["key"]
    
    # Ler arquivo
    obj = s3.get_object(Bucket=bucket, Key=key)
    df = pd.read_csv(obj["Body"])
    
    # Transformar
    df = df.dropna(subset=["id"]).drop_duplicates("id")
    df["processado_em"] = datetime.utcnow().isoformat()
    
    # Salvar processado
    buffer = StringIO()
    df.to_csv(buffer, index=False)
    s3.put_object(
        Bucket=bucket,
        Key=key.replace("raw/", "processed/"),
        Body=buffer.getvalue()
    )
    print(f"Arquivo processado: {key} ({len(df)} registros)")
```

### Lambda com Terraform
```hcl
# Empacotar o código
data "archive_file" "lambda_zip" {
  type        = "zip"
  source_dir  = "${path.module}/lambda_code/"
  output_path = "${path.module}/lambda.zip"
}

# Criar a função Lambda
resource "aws_lambda_function" "processador" {
  function_name    = "processador-eventos"
  filename         = data.archive_file.lambda_zip.output_path
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
  runtime          = "python3.12"
  handler          = "lambda_function.lambda_handler"
  timeout          = 300    # 5 minutos máximo
  memory_size      = 512    # MB

  role = aws_iam_role.lambda_role.arn

  environment {
    variables = {
      ENV         = "prod"
      BUCKET_NAME = aws_s3_bucket.dados.bucket
    }
  }
}

# Trigger: SQS → Lambda
resource "aws_lambda_event_source_mapping" "sqs_trigger" {
  event_source_arn                   = aws_sqs_queue.fila_eventos.arn
  function_name                      = aws_lambda_function.processador.arn
  batch_size                         = 10       # processa até 10 msgs por vez
  maximum_batching_window_in_seconds = 30       # aguarda até 30s para acumular
  function_response_types            = ["ReportBatchItemFailures"]
}
```

---

## Padrão Completo: Ingestão Event-Driven

```
[S3: arquivo novo]
       ↓ S3 Event Notification
[SQS: fila de ingestão]
       ↓ trigger
[Lambda: processar arquivo]
       ↓ sucesso
[SQS: fila de transformação]
       ↓ trigger
[Lambda: transformar + validar]
       ↓ falha
[SQS: Dead Letter Queue (DLQ)]
       ↓
[Lambda: notificar time via Slack]
```

```python
# Publicar evento ao receber arquivo no S3
# (configurado como S3 Event Notification)

# Ou publicar manualmente de uma API
@app.post("/pedidos")
async def criar_pedido(pedido: PedidoSchema):
    # Salvar no banco
    id_pedido = await salvar_pedido(pedido)
    
    # Publicar evento para processamento assíncrono
    sqs.send_message(
        QueueUrl=QUEUE_PROCESSAMENTO,
        MessageBody=json.dumps({
            "tipo": "pedido_criado",
            "id_pedido": id_pedido,
            "timestamp": datetime.utcnow().isoformat()
        })
    )
    
    return {"id": id_pedido, "status": "recebido"}
```

---

## Dead Letter Queue (DLQ) — Capturar Falhas
```python
# Criar fila principal + DLQ
resource "aws_sqs_queue" "dlq" {
  name = "fila-eventos-dlq"
  message_retention_seconds = 1209600  # 14 dias
}

resource "aws_sqs_queue" "principal" {
  name = "fila-eventos"
  
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.dlq.arn
    maxReceiveCount     = 3   # após 3 falhas → vai para DLQ
  })
}

# Monitorar DLQ com CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "dlq_alarm" {
  alarm_name          = "mensagens-na-dlq"
  comparison_operator = "GreaterThanThreshold"
  threshold           = 0
  metric_name         = "ApproximateNumberOfMessagesVisible"
  namespace           = "AWS/SQS"
  dimensions = { QueueName = aws_sqs_queue.dlq.name }
  
  alarm_actions = [aws_sns_topic.alertas.arn]  # notifica via email/Slack
}
```

---

## ✅ Quando usar / ❌ Quando não usar

✅ USE Event-Driven quando:
- Processamento assíncrono é aceitável
- Desacoplamento entre sistemas é importante
- Picos de carga imprevisíveis (SQS absorve o burst)
- Múltiplos consumidores para o mesmo evento

❌ NÃO USE quando:
- Precisa de resposta síncrona (use REST API)
- Lógica simples demais (um cron job resolve)
- Time não tem familiaridade com AWS
- Budget é restrito para começar (use Airflow + Python)
