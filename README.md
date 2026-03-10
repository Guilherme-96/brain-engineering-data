# 🧠 Segundo Cérebro — Engenharia de Dados

> Base de conhecimento pessoal sobre Engenharia de Dados, construída como um vault [Obsidian](https://obsidian.md/) interligado — cada tecnologia é um nó, cada relação é uma aresta.

![Graph View](https://img.shields.io/badge/Obsidian-Graph%20View-7C3AED?style=flat&logo=obsidian&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat&logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

---

## 📸 Preview

> Abra no Obsidian e acesse a **Visualização em Gráfico** para ver a rede de conhecimento completa.

O gráfico mostra como todas as tecnologias se conectam — clicar em um nó revela os tópicos relacionados, criando um mapa mental interativo do ecossistema de dados.

---

## 🗂️ Conteúdo (49 arquivos)

### 🚀 Ponto de Partida
| Arquivo | Descrição |
|---------|-----------|
| [Minha_Jornada](Minha_Jornada.md) | Projetos pessoais, linha do tempo de evolução e próximos passos |
| [Escolha_de_Stack](Escolha_de_Stack.md) | Framework de decisão — tecnologia certa para o contexto certo |

### 🐍 Python e Bibliotecas
| Arquivo | Descrição |
|---------|-----------|
| [Python](Python.md) | Funções, POO, decoradores, generators, logging |
| [Pandas](Pandas.md) | Leitura, limpeza, transformação, merge, pivot, performance |
| [NumPy](NumPy.md) | Arrays, vetorização, broadcasting, estatísticas |
| [Requests](Requests.md) | HTTP, APIs REST, session, paginação, retry |
| [Playwright](Playwright.md) | Web scraping moderno, async, extração de tabelas |
| [Selenium](Selenium.md) | Automação de browsers, esperas explícitas |
| [SQLAlchemy](SQLAlchemy.md) | ORM, conexões, integração com Pandas |
| [OpenPyXL](OpenPyXL.md) | Leitura e escrita de Excel sem Office |
| [Pathlib](Pathlib.md) | Manipulação de caminhos e arquivos |
| [Itertools](Itertools.md) | Iteração eficiente, combinatórias, generators |

### 🏗️ Ambiente
| Arquivo | Descrição |
|---------|-----------|
| [Ambientes_Virtuais](Ambientes_Virtuais.md) | pyenv + venv + Poetry — fluxo recomendado |
| [pyenv](pyenv.md) | Gerenciar múltiplas versões do Python |
| [venv](venv.md) | Ambientes virtuais nativos |
| [Poetry](Poetry.md) | Gerenciamento moderno de dependências |
| [VSCode](VSCode.md) | Extensões, atalhos, settings.json recomendado |

### 🗄️ Bancos de Dados
| Arquivo | Descrição |
|---------|-----------|
| [PostgreSQL](PostgreSQL.md) | DDL, DML, Window Functions, CTEs, EXPLAIN |
| [MySQL](MySQL.md) | Diferenças do PostgreSQL, engines, charset |
| [SQLServer](SQLServer.md) | T-SQL, Stored Procedures, schemas |
| [MongoDB](MongoDB.md) | PyMongo, Aggregation Pipeline, índices |
| [Modelagem_de_Dados](Modelagem_de_Dados.md) | Normalização, Star Schema, SCD, índices |

### 🔀 Processamento e Orquestração
| Arquivo | Descrição |
|---------|-----------|
| [Airflow](Airflow.md) | DAGs, Operators, XCom, cron, Docker |
| [Spark_PySpark](Spark_PySpark.md) | DataFrames, SQL, Window Functions, performance |
| [dbt](dbt.md) | Modelos SQL, testes, snapshots, materializations |
| [DuckDB_Multiengine](DuckDB_Multiengine.md) | Analytics local, multi-engine ETL, integração com dbt |
| [N8N](N8N.md) | Automação low-code, integração entre SaaS |

### 📨 Streaming e Eventos
| Arquivo | Descrição |
|---------|-----------|
| [Kafka](Kafka.md) | Producers, consumers, Faust, Spark Streaming |
| [Event_Driven_SQS_Lambda](Event_Driven_SQS_Lambda.md) | SQS, Lambda, DLQ, Terraform, padrões event-driven |

### ☁️ Cloud e Infraestrutura
| Arquivo | Descrição |
|---------|-----------|
| [AWS](AWS.md) | S3, Lambda, IAM, CLI, tabela de serviços |
| [Azure](Azure.md) | ADLS, Synapse, ADF, Databricks, Terraform |
| [Databricks](Databricks.md) | Delta Lake, arquitetura medalhão, Unity Catalog |
| [Docker](Docker.md) | Imagens, containers, Compose, volumes |
| [Terraform](Terraform.md) | IaC para AWS e Azure, módulos, state |
| [Deploys](Deploys.md) | CI/CD com GitHub Actions, ECS, Nginx, Blue-Green |

### 🌐 APIs e Apps
| Arquivo | Descrição |
|---------|-----------|
| [REST_API](REST_API.md) | Fundamentos HTTP, verbos, JWT, paginação |
| [FastAPI](FastAPI.md) | CRUD completo, Pydantic, auth JWT, Docker |
| [Streamlit](Streamlit.md) | Dashboard de dados com Python puro |

### 📊 Visualização
| Arquivo | Descrição |
|---------|-----------|
| [PowerBI](PowerBI.md) | Power Query, DAX, modelagem, calendário |
| [Metabase](Metabase.md) | SQL filters, API REST, self-hosted |
| [Tableau](Tableau.md) | LOD expressions, cálculos, TabPy |

### 🏛️ Qualidade e Governança
| Arquivo | Descrição |
|---------|-----------|
| [Governanca_de_Dados](Governanca_de_Dados.md) | Catálogo, lineage, LGPD, DataMesh |
| [Observabilidade_Pipelines](Observabilidade_Pipelines.md) | Logs, Prometheus, Great Expectations, alertas |

### 🧮 Fundamentos
| Arquivo | Descrição |
|---------|-----------|
| [Algoritmos_e_Estruturas_de_Dados](Algoritmos_e_Estruturas_de_Dados.md) | Big O, estruturas, técnicas de otimização |
| [Design_Patterns](Design_Patterns.md) | GoF em Python — Singleton, Factory, Observer, Strategy... |
| [Git_GitHub](Git_GitHub.md) | Fluxo completo, .gitignore, Conventional Commits, CI/CD |

### 💻 Sistemas Operacionais
| Arquivo | Descrição |
|---------|-----------|
| [Linux](Linux.md) | Terminal, permissões, processos, shell script |
| [Windows](Windows.md) | PowerShell, Chocolatey, WSL2 |

---

## 🚀 Como Usar

### 1. Clonar o repositório
```bash
git clone https://github.com/Guilherme-96/cerebro-engenharia-dados.git
```

### 2. Abrir no Obsidian
1. Baixe o [Obsidian](https://obsidian.md/) (gratuito)
2. Abra o Obsidian → **Abrir outro cofre** → **Abrir pasta como cofre**
3. Selecione a pasta clonada
4. Clique em **Visualização em gráfico** na barra lateral esquerda

### 3. Navegar
- Use o **gráfico** para visualizar conexões entre tecnologias
- Clique em qualquer `[[link]]` para navegar entre arquivos
- Use `Ctrl+P` para buscar qualquer tópico rapidamente
- Comece pelo arquivo [00_INDEX](00_INDEX.md) ou [Minha_Jornada](Minha_Jornada.md)

---

## 🤝 Contribuindo

Encontrou algum erro, quer adicionar uma tecnologia ou melhorar algum exemplo?

1. Fork o repositório
2. Crie uma branch: `git checkout -b feat/nova-tecnologia`
3. Faça suas alterações e commit: `git commit -m "feat: adiciona arquivo Kafka.md"`
4. Abra um Pull Request

---

## 📄 Licença

MIT — use, adapte e compartilhe à vontade.

---

## 👤 Autor

**Guilherme** — [@Guilherme-96](https://github.com/Guilherme-96)

> *"A melhor ferramenta é aquela que resolve o problema. A pior é a que você usou porque era a única que conhecia."*
