# 🎯 Escolha de Stack — Tecnologia Certa para o Contexto Certo

> "A melhor ferramenta é aquela que resolve o problema. A pior é a que você usou porque era a única que conhecia."

**Relacionado:** [[Modelagem_de_Dados]] | [[Docker]] | [[Airflow]] | [[PostgreSQL]] | [[MongoDB]] | [[Spark_PySpark]] | [[AWS]] | [[Azure]] | [[Databricks]] | [[DuckDB_Multiengine]]

---

## 🧭 Framework de Decisão

Antes de escolher qualquer tecnologia, responda:

1. **Qual o volume de dados?** (KB, MB, GB, TB, PB)
2. **Qual a frequência?** (tempo real, horário, diário, mensal)
3. **Quem vai consumir?** (analistas, sistemas, APIs, dashboards)
4. **Qual o budget?** (open source, cloud gerenciado, licença paga)
5. **Qual o tamanho do time?** (solo, startup, empresa grande)
6. **Qual o prazo?** (MVP em 1 semana ou sistema de produção)

---

## 📦 Banco de Dados — Quando Usar Cada Um

### SQL vs NoSQL — Decisão Base
```
Dados têm estrutura fixa e relações?          → SQL
Dados mudam de estrutura com frequência?      → NoSQL
Precisa de transações ACID complexas?         → SQL
Precisa escalar horizontalmente (sharding)?   → NoSQL
```

### Comparativo Detalhado

| Critério | PostgreSQL | MySQL | SQL Server | MongoDB |
|----------|-----------|-------|-----------|---------|
| Complexidade SQL | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ |
| JSON/Semi-estruturado | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Escala horizontal | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Ecosistema Python | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Custo open source | ✅ | ✅ | ❌ | ✅ |
| Melhor para... | Analytics | Web apps | Empresas MS | Docs flexíveis |

---

## ⚡ Processamento — Pandas vs Spark vs DuckDB

```
Volume de dados?

< 1 GB     → Pandas (simples, rápido, no notebook)
1 GB - 100 GB → DuckDB (SQL analítico, in-process, surpreendente)
> 100 GB   → Spark (distribuído, cluster) ou Databricks
Streaming? → Spark Streaming ou Kafka + Flink
```

### Pandas
✅ USE quando:
- Dados cabem na memória (< memória RAM)
- Exploração e análise interativa
- Prototipagem rápida
- Time sem conhecimento de Spark

❌ NÃO USE quando:
- Dados maiores que a RAM
- Precisa de processamento distribuído
- Performance é crítica em produção

### [[Spark_PySpark]] / [[Databricks]]
✅ USE quando:
- Dados em TB ou PB
- Processamento distribuído em cluster
- Integração com Delta Lake / Data Lakehouse
- Time já tem infraestrutura Hadoop/Spark

❌ NÃO USE quando:
- Dados são pequenos (overhead absurdo para < 1GB)
- Projeto solo sem cluster disponível
- Prototipagem rápida (setup é pesado)
- Budget é restrito

### [[DuckDB_Multiengine]] (DuckDB)
✅ USE quando:
- Analytics em arquivos Parquet/CSV locais (1-100GB)
- SQL moderno sem servidor
- Substituir Pandas em análises analíticas
- Integrar com dbt

❌ NÃO USE quando:
- Precisa de transações OLTP (é analítico)
- Dados são maiores que o disco local
- Precisa de concorrência alta de escrita

---

## 🌊 Orquestração — Airflow vs outros

### [[Airflow]]
✅ USE quando:
- Pipelines complexos com dependências
- Agendamento recorrente (diário, horário)
- Time técnico que entende Python
- Empresa com infraestrutura própria

❌ NÃO USE quando:
- Pipeline é simples demais (um script cron resolve)
- Time não tem Python
- Precisa de algo serverless/simples (use [[N8N]] ou Lambda)

### [[N8N]]
✅ USE quando:
- Integração entre SaaS (Slack + Google Sheets + Email)
- Time não-técnico precisa criar automações
- MVP rápido de workflow
- Low-code é suficiente

❌ NÃO USE quando:
- Transformações pesadas de dados
- Lógica complexa em Python/SQL
- Precisa de controle de versão de código

---

## ☁️ Cloud — AWS vs Azure vs GCP

### [[AWS]]
✅ USE quando:
- Maior ecossistema de serviços de dados
- Time já tem expertise em AWS
- Precisa de Redshift, EMR, Kinesis
- Budget para pagar um pouco mais

### [[Azure]]
✅ USE quando:
- Empresa já usa Microsoft (Office 365, SQL Server)
- Precisa de integração com Power BI
- Azure Synapse Analytics é o requisito
- Time já usa Azure DevOps

❌ NÃO USE AWS ou Azure quando:
- Budget é muito restrito → use VPS + Docker
- Projeto pessoal/aprendizado → use free tier + Terraform para controlar custos

---

## 🏗️ Containerização e Deploy

### [[Docker]] local vs Cloud
```
Desenvolvimento local    → Docker Compose
Produção simples         → Docker + VPS (EC2, Azure VM)
Produção escalável       → Kubernetes (EKS, AKS, GKE)
Serverless               → Lambda, Azure Functions, Cloud Run
```

### Quando NÃO containerizar
- Scripts de análise exploratória (notebook é suficiente)
- Projetos pessoais descartáveis
- Times que não têm familiaridade (curva de aprendizado alta)

---

## 📊 Visualização — Quando Usar Cada Um

| Ferramenta | Use quando... | Não use quando... |
|------------|--------------|-------------------|
| [[PowerBI]] | Empresa usa Microsoft, analistas de negócio | Precisa de embed em app, budget apertado |
| [[Tableau]] | Visualizações complexas, empresa grande | Budget restrito, equipe pequena |
| [[Metabase]] | Time técnico, open source, SQL direto | Precisa de DAX/MDX avançado |
| [[Streamlit]] | Protótipo de app de dados, Python puro | Dashboard corporativo com muitos usuários |
| Plotly/Dash | App interativo customizado em Python | Analistas de negócio sem código |

---

## 🔄 ETL vs ELT

```
ETL (Extract → Transform → Load)
- Transforma ANTES de carregar
- Ideal para: dados sensíveis, destinos legados, transformações pesadas

ELT (Extract → Load → Transform)
- Carrega RAW, transforma no destino (dbt, Spark)
- Ideal para: cloud data warehouses (Redshift, BigQuery, Snowflake)
- Moderno e escalável
```

---

## 🎓 Princípio Final

> **"Não existe stack perfeita. Existe stack adequada para o contexto."**

Um pipeline com `pandas + cron + PostgreSQL` que funciona em produção há 2 anos **é melhor** que um pipeline com Spark + Airflow + Kubernetes que nunca foi ao ar.

Comece simples. Escale quando o problema aparecer.
