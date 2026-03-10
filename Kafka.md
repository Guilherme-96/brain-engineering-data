# 📨 Apache Kafka

> Plataforma de streaming distribuído. Processa milhões de eventos por segundo com garantia de entrega.

**Relacionado:** [[Event_Driven_SQS_Lambda]] | [[AWS]] | [[Docker]] | [[Python]] | [[Spark_PySpark]] | [[Airflow]] | [[Escolha_de_Stack]]

---

## Kafka vs SQS

| Critério | Kafka | Amazon SQS |
|---------|-------|-----------|
| Throughput | Milhões/seg | Milhares/seg |
| Retenção | Configurável (dias/infinito) | 14 dias máximo |
| Replay | ✅ Reprocessar histórico | ❌ Mensagem some após consumo |
| Múltiplos consumidores | ✅ Consumer Groups | ✅ Mas cada msg é consumida 1x |
| Ordenação | Por partição | FIFO Queue (limitada) |
| Setup | Complexo (ZooKeeper/KRaft) | Gerenciado pela AWS |
| Custo | Infra própria ou Confluent | Pague por mensagem |
| Ideal para | Streaming, logs, event sourcing | Desacoplamento simples, serverless |

---

## Conceitos Fundamentais

```
Produtor → publica em → Tópico → dividido em → Partições
                                                     ↓
                                              Consumidores (Consumer Group)

Tópico = categoria/feed de mensagens
Partição = unidade de paralelismo e ordenação
Offset = posição da mensagem na partição
Consumer Group = grupo de consumidores que divide as partições
Broker = servidor Kafka
```

---

## Instalação com Docker
```yaml
# docker-compose.yml
services:
  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"
    ports:
      - "9092:9092"

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    depends_on:
      - kafka
```

---

## Produzir e Consumir com Python
```python
# pip install confluent-kafka

from confluent_kafka import Producer, Consumer
import json
from datetime import datetime

BOOTSTRAP_SERVERS = "localhost:9092"
TOPICO = "eventos-vendas"

# PRODUTOR
producer = Producer({"bootstrap.servers": BOOTSTRAP_SERVERS})

def publicar_evento(tipo: str, dados: dict):
    payload = {
        "tipo": tipo,
        "timestamp": datetime.utcnow().isoformat(),
        **dados
    }
    
    producer.produce(
        topic=TOPICO,
        key=str(dados.get("id", "")),   # chave define a partição (mesma key → mesma partição → ordenado)
        value=json.dumps(payload).encode("utf-8"),
        on_delivery=lambda err, msg: print(f"✅ {msg.topic()}[{msg.partition()}]" if not err else f"❌ {err}")
    )
    producer.flush()   # aguardar confirmação

# Publicar
publicar_evento("pedido_criado", {"id": 123, "valor": 150.00, "cliente_id": 456})
publicar_evento("pedido_aprovado", {"id": 123, "gateway": "stripe"})


# CONSUMIDOR
consumer = Consumer({
    "bootstrap.servers": BOOTSTRAP_SERVERS,
    "group.id": "pipeline-processador",     # Consumer Group
    "auto.offset.reset": "earliest",        # começar do início se novo grupo
    "enable.auto.commit": False             # commit manual (mais seguro)
})

consumer.subscribe([TOPICO])

try:
    while True:
        msg = consumer.poll(timeout=1.0)
        
        if msg is None:
            continue
        if msg.error():
            print(f"Erro: {msg.error()}")
            continue
        
        evento = json.loads(msg.value().decode("utf-8"))
        
        try:
            processar_evento(evento)
            consumer.commit(msg)   # só confirma se processou com sucesso
        except Exception as e:
            print(f"Erro ao processar {evento['tipo']}: {e}")
            # Não faz commit → mensagem será reprocessada

finally:
    consumer.close()
```

---

## Kafka Streams com Python (Faust)
```python
# pip install faust-streaming
import faust
from datetime import datetime

app = faust.App("pipeline-vendas", broker="kafka://localhost:9092")

class EventoVenda(faust.Record):
    id: int
    valor: float
    cliente_id: int
    timestamp: str

class VendasAgregadas(faust.Record):
    hora: str
    total: float
    qtd: int

topico_entrada = app.topic("eventos-vendas", value_type=EventoVenda)
topico_saida = app.topic("vendas-agregadas", value_type=VendasAgregadas)

# Tabela de estado (mantida em memória + changelog no Kafka)
tabela_totais = app.Table("totais-por-hora", default=float)

@app.agent(topico_entrada)
async def processar(vendas):
    async for venda in vendas:
        hora = venda.timestamp[:13]   # "2024-01-15T08"
        tabela_totais[hora] += venda.valor
        
        print(f"Hora {hora}: R$ {tabela_totais[hora]:.2f}")

# Iniciar: faust -A main worker
```

---

## Kafka com Spark Structured Streaming
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder \
    .appName("KafkaStream") \
    .config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0") \
    .getOrCreate()

# Ler do Kafka como stream
df_kafka = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "eventos-vendas") \
    .option("startingOffsets", "latest") \
    .load()

# Parsear JSON
schema = "id INT, valor DOUBLE, cliente_id INT, timestamp STRING"
df_parsed = df_kafka \
    .select(F.from_json(F.col("value").cast("string"), schema).alias("data")) \
    .select("data.*") \
    .withColumn("timestamp", F.to_timestamp("timestamp"))

# Agregar por janela de 5 minutos
df_agregado = df_parsed \
    .withWatermark("timestamp", "2 minutes") \
    .groupBy(F.window("timestamp", "5 minutes")) \
    .agg(F.sum("valor").alias("total"), F.count("*").alias("qtd"))

# Escrever resultado
query = df_agregado.writeStream \
    .outputMode("update") \
    .format("console") \
    .option("truncate", False) \
    .start()

query.awaitTermination()
```

---

## ✅ Quando usar Kafka / ❌ Quando não usar

✅ USE Kafka quando:
- Streaming de eventos em tempo real (IoT, logs, clickstream)
- Múltiplos sistemas consumindo o mesmo evento
- Precisa reprocessar histórico de eventos (event sourcing)
- Alto throughput (>10k eventos/segundo)
- Desacoplamento entre muitos sistemas

❌ NÃO USE Kafka quando:
- Filas simples entre 2 sistemas → use [[Event_Driven_SQS_Lambda]]
- Pipeline batch (não precisa de streaming) → use [[Airflow]]
- Budget restrito (Kafka é caro para manter) → use SQS
- Time sem expertise em Kafka (curva de aprendizado alta)
