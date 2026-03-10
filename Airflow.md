# 🌊 Apache Airflow

> Plataforma de orquestração de pipelines de dados. Define, agenda e monitora workflows.

**Relacionado:** [[Python]] | [[Docker]] | [[PostgreSQL]] | [[AWS]] | [[Spark_PySpark]] | [[Git_GitHub]]

---

## Conceitos Fundamentais
- **DAG** (Directed Acyclic Graph): define o fluxo do pipeline — as tasks e suas dependências
- **Task:** unidade mínima de trabalho
- **Operator:** template de task (PythonOperator, BashOperator, etc.)
- **Scheduler:** processo que monitora e dispara DAGs
- **Webserver:** interface web para monitorar e gerenciar
- **Executor:** como as tasks são executadas (LocalExecutor, CeleryExecutor, KubernetesExecutor)
- **XCom:** mecanismo de troca de dados entre tasks
- **Connection:** credenciais armazenadas centralmente
- **Variable:** configurações globais

---

## Instalação
```bash
# Instalar (use a versão do Airflow compatível com seu Python)
pip install 'apache-airflow==2.10.3' \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.10.3/constraints-3.12.txt"

# Inicializar banco de dados
export AIRFLOW_HOME=~/airflow
airflow db init

# Criar usuário admin
airflow users create \
  --username admin --password admin \
  --firstname Admin --lastname User \
  --role Admin --email admin@example.com

# Subir Airflow (tudo em um — para dev)
airflow standalone
# Interface: http://localhost:8080
```

---

## Estrutura de Pastas
```
$AIRFLOW_HOME/
├── dags/           # seus DAGs ficam aqui
├── logs/           # logs de execução
├── plugins/        # operators e hooks customizados
└── airflow.cfg     # configuração
```

---

## Criando DAGs
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.operators.empty import EmptyOperator

default_args = {
    "owner": "guiga96",
    "depends_on_past": False,
    "email_on_failure": False,
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "execution_timeout": timedelta(hours=1),
}

with DAG(
    dag_id="pipeline_vendas",
    description="Pipeline de ingestão de dados de vendas",
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule="0 6 * * *",     # todo dia às 6h
    catchup=False,             # não executar datas passadas
    max_active_runs=1,
    tags=["vendas", "producao"],
) as dag:

    inicio = EmptyOperator(task_id="inicio")
    fim = EmptyOperator(task_id="fim")

    def extrair(**context):
        data = context["ds"]    # data de execução: YYYY-MM-DD
        print(f"Extraindo dados de {data}")
        return {"registros": 1000}  # vai para XCom automaticamente

    def transformar(**context):
        ti = context["ti"]
        dados = ti.xcom_pull(task_ids="extrair")
        print(f"Transformando {dados['registros']} registros")

    task_extrair = PythonOperator(
        task_id="extrair",
        python_callable=extrair,
    )

    task_transformar = PythonOperator(
        task_id="transformar",
        python_callable=transformar,
    )

    task_bash = BashOperator(
        task_id="notificar",
        bash_command='echo "Pipeline concluído em {{ ds }}"',
    )

    # Dependências
    inicio >> task_extrair >> task_transformar >> task_bash >> fim

    # Paralelismo
    # inicio >> [task_a, task_b] >> task_c >> fim
```

---

## Expressões Cron
```
┌── minuto (0-59)
│ ┌── hora (0-23)
│ │ ┌── dia do mês (1-31)
│ │ │ ┌── mês (1-12)
│ │ │ │ ┌── dia da semana (0=dom, 6=sáb)
│ │ │ │ │
* * * * *

"0 6 * * *"          # todo dia às 6h
"0 6 * * 1-5"        # 6h de seg a sex
"0 */4 * * *"        # a cada 4 horas
"0 0 1 * *"          # todo dia 1º do mês
"*/15 * * * *"       # a cada 15 minutos
"0 8,12,18 * * *"    # às 8h, 12h e 18h

# Atalhos
@daily   = "0 0 * * *"
@hourly  = "0 * * * *"
@weekly  = "0 0 * * 0"
@monthly = "0 0 1 * *"
```

---

## Operators Principais
```python
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.bash import BashOperator
from airflow.operators.empty import EmptyOperator
from airflow.operators.email import EmailOperator
from airflow.sensors.filesystem import FileSensor
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.amazon.aws.operators.s3 import S3CopyObjectOperator

# BranchPythonOperator — desvio condicional
def escolher_branch(**context):
    if condition:
        return "task_a"    # task_id para executar
    return "task_b"

branch = BranchPythonOperator(
    task_id="decisao",
    python_callable=escolher_branch
)

# Sensor — aguarda condição externa
aguardar_arquivo = FileSensor(
    task_id="aguardar_arquivo",
    filepath="/data/entrada/arquivo.csv",
    poke_interval=60,       # checar a cada 60s
    timeout=3600,           # timeout em 1h
    mode="reschedule"       # libera worker enquanto aguarda
)

# PostgresOperator — executar SQL
criar_tabela = PostgresOperator(
    task_id="criar_tabela",
    postgres_conn_id="postgres_default",
    sql="CREATE TABLE IF NOT EXISTS vendas (...)"
)
```

---

## XCom — Comunicação entre Tasks
```python
# Enviar via return (automático)
def extrair():
    return {"total": 1000}

# Enviar explicitamente
def processar(**context):
    ti = context["ti"]
    ti.xcom_push(key="minha_chave", value={"resultado": "ok"})

# Receber
def usar_resultado(**context):
    ti = context["ti"]
    
    # Pegar return da task
    dados = ti.xcom_pull(task_ids="extrair")
    
    # Pegar por chave específica
    resultado = ti.xcom_pull(task_ids="processar", key="minha_chave")

# No template Jinja (BashOperator, SQL, etc.)
# "{{ ti.xcom_pull(task_ids='extrair')['total'] }}"
```

---

## Connections — Configurar Credenciais
```bash
# Via CLI
airflow connections add postgres_default \
  --conn-type postgres \
  --conn-host localhost \
  --conn-login admin \
  --conn-password senha123 \
  --conn-port 5432 \
  --conn-schema meudb

# Via interface: Admin → Connections
```

---

## Airflow com Docker
```yaml
# docker-compose.yml mínimo
version: '3.9'
services:
  airflow:
    image: apache/airflow:2.10.3-python3.12
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    command: standalone
```
