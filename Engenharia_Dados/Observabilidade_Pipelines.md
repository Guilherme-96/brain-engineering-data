# 🔭 Observabilidade em Pipelines de Dados

> Saber que seu pipeline está funcionando ANTES do usuário reclamar. Logs, métricas e alertas.

**Relacionado:** [[Airflow]] | [[Docker]] | [[AWS]] | [[Azure]] | [[FastAPI]] | [[dbt]] | [[Governanca_de_Dados]] | [[Python]]

---

## Os Três Pilares

```
Logs      → "O que aconteceu?" (registros de eventos)
Métricas  → "Quão saudável está?" (números no tempo)
Traces    → "Por onde o dado passou?" (caminho de execução)
```

---

## 1. Logs Estruturados em Python
```python
import logging
import json
from datetime import datetime
from typing import Any

# Logger estruturado (JSON) — muito mais fácil de filtrar
class JsonLogger:
    def __init__(self, pipeline: str):
        self.pipeline = pipeline
        self.logger = logging.getLogger(pipeline)
        handler = logging.StreamHandler()
        handler.setFormatter(logging.Formatter('%(message)s'))
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.INFO)

    def _log(self, level: str, evento: str, **kwargs):
        entrada = {
            "timestamp": datetime.utcnow().isoformat(),
            "pipeline": self.pipeline,
            "nivel": level,
            "evento": evento,
            **kwargs
        }
        getattr(self.logger, level.lower())(json.dumps(entrada))

    def inicio(self, **kwargs): self._log("INFO", "pipeline_iniciado", **kwargs)
    def fim(self, **kwargs): self._log("INFO", "pipeline_concluido", **kwargs)
    def erro(self, erro: Exception, **kwargs): 
        self._log("ERROR", "pipeline_erro", erro=str(erro), tipo=type(erro).__name__, **kwargs)
    def metrica(self, nome: str, valor: Any, **kwargs):
        self._log("INFO", "metrica", metrica=nome, valor=valor, **kwargs)


# Uso no pipeline
log = JsonLogger("pipeline_vendas")

def extrair_dados(data: str):
    log.inicio(data=data, fonte="postgresql")
    try:
        inicio = datetime.utcnow()
        # ... lógica de extração
        df = pd.read_sql(query, engine)
        duracao = (datetime.utcnow() - inicio).total_seconds()
        
        log.metrica("registros_extraidos", len(df), data=data)
        log.metrica("duracao_extracao_seg", duracao)
        return df
    except Exception as e:
        log.erro(e, data=data, etapa="extracao")
        raise

# Saída JSON:
# {"timestamp":"2024-01-15T08:00:01","pipeline":"pipeline_vendas","nivel":"INFO","evento":"metrica","metrica":"registros_extraidos","valor":15320,"data":"2024-01-14"}
```

---

## 2. Métricas com Prometheus + Grafana

```python
# pip install prometheus-client
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Definir métricas
registros_processados = Counter(
    'pipeline_registros_total',
    'Total de registros processados',
    ['pipeline', 'status']  # labels
)

duracao_pipeline = Histogram(
    'pipeline_duracao_segundos',
    'Duração do pipeline em segundos',
    ['pipeline'],
    buckets=[1, 5, 10, 30, 60, 120, 300, 600]
)

registros_ativos = Gauge(
    'pipeline_registros_processando',
    'Registros sendo processados agora',
    ['pipeline']
)

erros_validacao = Counter(
    'pipeline_erros_validacao_total',
    'Total de erros de validação',
    ['pipeline', 'campo']
)

# Usar no código
start_http_server(8000)  # expõe métricas em /metrics

with duracao_pipeline.labels('vendas').time():
    registros_ativos.labels('vendas').set(len(df))
    
    for i, lote in enumerate(chunks(df, 1000)):
        processar_lote(lote)
        registros_processados.labels('vendas', 'sucesso').inc(len(lote))
    
    registros_ativos.labels('vendas').set(0)
```

```yaml
# docker-compose.yml para stack de observabilidade
services:
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin

  # Seu pipeline expondo métricas
  pipeline:
    build: .
    ports: ["8000:8000"]
```

---

## 3. Great Expectations — Qualidade de Dados
```python
import great_expectations as gx
import pandas as pd

# Configurar contexto
context = gx.get_context()

# Criar expectativas para o DataFrame
validator = context.sources.pandas_default.read_dataframe(
    dataframe=df,
    asset_name="vendas_diarias"
)

# Regras de qualidade
validator.expect_column_values_to_not_be_null("id_pedido")
validator.expect_column_values_to_be_unique("id_pedido")
validator.expect_column_values_to_not_be_null("data_pedido")
validator.expect_column_values_to_be_between(
    column="valor",
    min_value=0.01,
    max_value=1_000_000
)
validator.expect_column_values_to_be_in_set(
    column="status",
    value_set=["APROVADO", "PENDENTE", "CANCELADO"]
)
validator.expect_column_values_to_match_regex(
    column="cpf",
    regex=r"^\d{11}$"
)
validator.expect_table_row_count_to_be_between(
    min_value=100,    # Se chegarem menos de 100 registros, algo está errado
    max_value=100_000
)

# Validar
resultado = validator.validate()

if not resultado.success:
    falhas = [r for r in resultado.results if not r.success]
    for falha in falhas:
        print(f"❌ FALHA: {falha.expectation_config.expectation_type}")
        print(f"   Coluna: {falha.expectation_config.kwargs.get('column')}")
    raise ValueError(f"Validação de dados falhou! {len(falhas)} expectativas não atendidas.")

print(f"✅ Validação passou! {len(resultado.results)} expectativas verificadas.")
```

---

## 4. Alertas no Airflow
```python
from airflow.operators.python import PythonOperator
from airflow.models import Variable
import requests

def alertar_slack(context):
    """Callback executado quando uma task falha."""
    dag_id = context['dag'].dag_id
    task_id = context['task_instance'].task_id
    exec_date = context['execution_date']
    log_url = context['task_instance'].log_url
    
    webhook_url = Variable.get("SLACK_WEBHOOK_URL")
    
    mensagem = {
        "text": f"🚨 *Pipeline com falha!*",
        "attachments": [{
            "color": "danger",
            "fields": [
                {"title": "DAG", "value": dag_id, "short": True},
                {"title": "Task", "value": task_id, "short": True},
                {"title": "Data", "value": str(exec_date)[:19], "short": True},
                {"title": "Logs", "value": f"<{log_url}|Ver logs>", "short": True}
            ]
        }]
    }
    requests.post(webhook_url, json=mensagem)

# Configurar na DAG
default_args = {
    "on_failure_callback": alertar_slack,
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
}
```

---

## 5. Dashboard de Saúde do Pipeline

```python
# app_monitoramento.py com Streamlit
import streamlit as st
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://...")

st.title("🔭 Saúde dos Pipelines")

# Execuções recentes
df_exec = pd.read_sql("""
    SELECT 
        dag_id, state, 
        execution_date,
        end_date - start_date as duracao
    FROM dag_run 
    WHERE execution_date > NOW() - INTERVAL '7 days'
    ORDER BY execution_date DESC
""", engine)

col1, col2, col3 = st.columns(3)
col1.metric("✅ Sucesso (7d)", len(df_exec[df_exec.state=='success']))
col2.metric("❌ Falhas (7d)", len(df_exec[df_exec.state=='failed']))
taxa = len(df_exec[df_exec.state=='success']) / len(df_exec) * 100
col3.metric("📊 Taxa de Sucesso", f"{taxa:.1f}%")

st.dataframe(df_exec)
```

---

## Stack Recomendada por Porte

| Porte | Logs | Métricas | Alertas | Qualidade |
|-------|------|----------|---------|-----------|
| Solo/Startup | Python logging + arquivo | Nenhuma/Básica | Email | dbt tests |
| Médio | ELK Stack ou Loki | Prometheus + Grafana | Slack/PagerDuty | Great Expectations |
| Enterprise | Datadog / New Relic | Datadog | PagerDuty | Monte Carlo / Soda |
