# 🔄 N8N — Automação de Workflows

> Plataforma de automação low-code/no-code. Conecta apps, APIs e serviços sem escrever muito código.

**Relacionado:** [[Airflow]] | [[REST_API]] | [[FastAPI]] | [[AWS]] | [[Python]] | [[Escolha_de_Stack]]

---

## O que é N8N?

```
N8N = "Nodemation" (Node + Automation)

Pense como: Zapier / Make (antigo Integromat), mas:
  ✅ Open source
  ✅ Self-hosted (seus dados ficam com você)
  ✅ Suporta código JavaScript/Python nos nodes
  ✅ Grátis para self-hosted
  ✅ Mais de 400 integrações nativas
```

---

## Instalação
```bash
# Docker (recomendado)
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=senha123 \
  n8nio/n8n

# Acessar: http://localhost:5678

# Ou npm
npm install -g n8n
n8n start
```

---

## Conceitos Fundamentais

### Nodes
Cada "bloco" no canvas é um node. Tipos principais:
- **Trigger nodes:** iniciam o workflow (Webhook, Schedule, Email, etc.)
- **Action nodes:** executam ações (HTTP Request, banco de dados, Slack, etc.)
- **Logic nodes:** controlam fluxo (IF, Switch, Merge, Loop)
- **Code node:** JavaScript ou Python puro

### Workflows
Conjunto de nodes conectados que formam uma automação.

---

## Casos de Uso em Dados

### 1. Pipeline de Notificação de Dados
```
[Schedule: todo dia 8h]
        ↓
[PostgreSQL: SELECT vendas do dia]
        ↓
[IF: vendas < meta]
        ↓ sim
[Slack: 🚨 Meta não atingida]
        ↓ não
[Slack: ✅ Meta atingida!]
```

### 2. Ingestão de API Externa
```
[Schedule: a cada hora]
        ↓
[HTTP Request: GET api.externa.com/dados]
        ↓
[Code node: transformar JSON]
        ↓
[PostgreSQL: INSERT dados transformados]
        ↓
[Email: relatório diário]
```

### 3. Webhook para Trigger de Pipeline
```
[Webhook: POST /trigger-pipeline]
        ↓
[Code node: validar payload]
        ↓
[HTTP Request: POST airflow/api/v1/dags/meu_dag/dagRuns]
        ↓
[Slack: Pipeline iniciado ✅]
```

---

## Code Node — Quando Low-Code Não Basta
```javascript
// Code node em JavaScript
const items = $input.all();

return items.map(item => {
  const dados = item.json;
  
  return {
    json: {
      id: dados.id,
      valor_formatado: `R$ ${(dados.valor / 100).toFixed(2)}`,
      data_br: new Date(dados.data).toLocaleDateString('pt-BR'),
      categoria: dados.valor > 1000 ? 'alto' : 'baixo'
    }
  };
});
```

```python
# Code node em Python
import json

items = $input.all()
resultado = []

for item in items:
    dados = item['json']
    resultado.append({
        'json': {
            'nome_upper': dados['nome'].upper(),
            'email_valido': '@' in dados.get('email', ''),
            'processado_em': __import__('datetime').datetime.now().isoformat()
        }
    })

return resultado
```

---

## Integrações Úteis para Dados

| Categoria | Integrações disponíveis |
|-----------|------------------------|
| Bancos de dados | PostgreSQL, MySQL, MongoDB, Redis, SQLite |
| Cloud | AWS S3, Google Drive, Azure Blob |
| Comunicação | Slack, Teams, Email, Telegram, WhatsApp |
| Produtividade | Google Sheets, Notion, Airtable, Trello |
| APIs | HTTP Request (qualquer API REST) |
| Mensageria | RabbitMQ, Kafka, Amazon SQS |
| Auth | OAuth2, JWT, API Key |

---

## N8N vs Airflow

| Critério | N8N | Airflow |
|---------|-----|---------|
| Curva de aprendizado | Baixa (visual) | Alta (Python) |
| Transformações pesadas | ❌ | ✅ |
| Integrações SaaS | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Versionamento de código | Limitado | Total (Python + Git) |
| Monitoramento | Básico | Avançado |
| Ideal para | Integração entre apps | Pipelines de dados |

---

## ✅ Quando usar N8N / ❌ Quando não usar

✅ USE N8N quando:
- Integrar sistemas SaaS (Slack + Sheets + Email + Notion)
- Automações de negócio sem transformações pesadas
- Time não-técnico precisa criar e manter workflows
- MVP rápido de automação (horas, não dias)
- Webhooks e triggers simples

❌ NÃO USE N8N quando:
- Transformações complexas de dados em Python/SQL
- Pipelines com dependências complexas
- Precisa de retry, backfill histórico, DAGs complexas → use [[Airflow]]
- Volume de dados alto → use [[Spark_PySpark]] ou [[Databricks]]
