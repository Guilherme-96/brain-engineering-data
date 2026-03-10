# 🗺️ Minha Jornada — Projetos e Evolução

> Registro da minha evolução como Engenheiro de Dados através dos projetos construídos.
> GitHub: https://github.com/Guilherme-96

---

## 📈 Linha do Tempo de Evolução

```
Automação Python
      ↓
Análise de Dados + Pandas
      ↓
Bancos de Dados (SQL + NoSQL)
      ↓
Pipelines de Dados
      ↓
Engenharia de Dados (stack completa)
      ↓
[próximo nível → Cloud + Orquestração + Observabilidade]
```

---

## 🚀 Projetos Públicos

### 1. 🛒 automacao-pesquisa-precos
**Stack:** [[Python]] | [[Playwright]] ou [[Selenium]] | [[Pandas]] | [[OpenPyXL]]

**O que faz:**
Automação de coleta de preços em sites de e-commerce. Acessa páginas web automaticamente, extrai preços de produtos e consolida em planilha ou banco de dados para acompanhamento.

**Conceitos aplicados:**
- Web scraping com controle de browser
- Tratamento de dados coletados com [[Pandas]]
- Exportação para Excel com [[OpenPyXL]]
- Uso de [[Pathlib]] para gerenciamento de arquivos

**Próximos passos possíveis:**
- Orquestrar com [[Airflow]] para rodar diariamente às 8h
- Armazenar histórico em [[PostgreSQL]] para análise de variação de preços
- Criar dashboard de acompanhamento com [[Streamlit]] ou [[Metabase]]

---

### 2. 📊 automacao-indicadores-lojas
**Stack:** [[Python]] | [[Pandas]] | [[OpenPyXL]] | [[Pathlib]]

**O que faz:**
Automação do cálculo de indicadores de performance de lojas (One Page). Lê arquivos de vendas, calcula KPIs (faturamento, diversidade de produtos, ticket médio) e envia relatório por email para cada gerente de loja.

**Conceitos aplicados:**
- ETL simples: leitura → transformação → output
- Agregações e cálculos com [[Pandas]]
- Automação de envio de emails (`smtplib`)
- Gerenciamento de pastas por loja com [[Pathlib]]
- [[Modelagem_de_Dados]] — entendimento de KPIs

**Por que este projeto é poderoso no portfólio:**
Demonstra um caso de uso real de negócio — automatizar um processo manual que tomava horas de trabalho diário de uma equipe.

**Próximos passos possíveis:**
- Substituir envio por email por dashboard no [[PowerBI]] ou [[Streamlit]]
- Adicionar banco de dados [[PostgreSQL]] para histórico
- Agendar no [[Airflow]]

---

### 3. 🗄️ pipeline_de_dados-mongodb-e-mysql
**Stack:** [[Python]] | [[MongoDB]] | [[MySQL]] | [[SQLAlchemy]] | [[Pandas]] | [[Docker]]

**O que faz:**
Pipeline de dados que integra dois tipos de banco — um NoSQL ([[MongoDB]]) e um relacional ([[MySQL]]). Provavelmente extrai dados de uma fonte, transforma e carrega em ambos os bancos conforme a natureza dos dados.

**Conceitos aplicados:**
- Arquitetura multi-banco (SQL + NoSQL)
- ETL com [[Python]] e [[Pandas]]
- Conexão com bancos via [[SQLAlchemy]] e PyMongo
- [[Modelagem_de_Dados]] — saber quando usar cada banco

**Decisão de arquitetura demonstrada:**
> "Usei MongoDB para dados flexíveis/semi-estruturados e MySQL para dados relacionais com integridade referencial"

Isso demonstra **maturidade técnica** — saber escolher a ferramenta certa. Ver [[Escolha_de_Stack]].

**Próximos passos:**
- Containerizar com [[Docker]] Compose
- Adicionar [[Observabilidade_Pipelines]]
- Orquestrar com [[Airflow]]

---

### 4. 🐙 linguagens-repositorios-empresas
**Stack:** [[Python]] | [[Requests]] | [[Pandas]] | API do GitHub

**O que faz:**
Consome a API pública do GitHub para analisar quais linguagens de programação as empresas mais usam em seus repositórios. Demonstra consumo de API REST e análise de dados.

**Conceitos aplicados:**
- Consumo de [[REST_API]] com [[Requests]]
- Paginação de API
- Análise e visualização com [[Pandas]]
- Tratamento de rate limit da API

**Por que é valioso:**
Demonstra capacidade de consumir APIs externas — habilidade central em pipelines de ingestão de dados.

**Próximos passos:**
- Transformar em pipeline com [[Airflow]]
- Criar [[FastAPI]] para servir os dados coletados
- Dashboard com [[Streamlit]]

---

### 5. 📁 organizar-downloads
**Stack:** [[Python]] | [[Pathlib]]

**O que faz:**
Script de automação que organiza automaticamente a pasta de Downloads, movendo arquivos para subpastas por tipo (imagens, documentos, vídeos, etc.).

**Conceitos aplicados:**
- Manipulação de sistema de arquivos com [[Pathlib]]
- Automação de tarefas repetitivas
- [[Linux]] e [[Windows]] cross-platform

**Por que é valioso:**
Simples mas demonstra o mindset de engenheiro — automatizar o que é repetitivo.

---

### 6. 🎨 Design-Patterns
**Stack:** [[Python]]

**O que faz:**
Implementações dos principais padrões de projeto GoF (Gang of Four) em Python — Singleton, Factory, Observer, Strategy, Decorator, etc.

**Conceitos aplicados:**
- Orientação a Objetos avançada (ver [[Python]])
- [[Algoritmos_e_Estruturas_de_Dados]]
- Código limpo e manutenível

**Por que é valioso:**
Design Patterns são o vocabulário comum de engenheiros sênior. Quem conhece patterns escreve código que outros engenheiros conseguem manter. Ver [[Design_Patterns]].

---

## 🎯 O que seus projetos revelam sobre você

### Pontos fortes identificados:
✅ **Automação** — múltiplos projetos de automação de processos reais
✅ **Multi-stack** — sabe trabalhar com SQL e NoSQL
✅ **ETL** — construiu pipelines do zero
✅ **Consumo de API** — integração com sistemas externos
✅ **Fundamentos sólidos** — Design Patterns demonstra preocupação com qualidade de código

### Gaps para fechar (próximos projetos):
🎯 Cloud — projeto usando [[AWS]] ou [[Azure]]
🎯 Orquestração — DAG no [[Airflow]] orquestrando seus pipelines
🎯 Containerização — seus projetos rodando em [[Docker]]
🎯 Observabilidade — monitoramento com [[Observabilidade_Pipelines]]
🎯 API própria — criar uma [[FastAPI]] para servir seus dados

---

## 💡 Projeto Sugerido para Fechar os Gaps

### "Pipeline de Preços com Stack Completa"

```
[Playwright scraping] 
      ↓
[PostgreSQL raw layer]  
      ↓
[dbt transformations]   
      ↓
[FastAPI endpoint]      
      ↓
[Streamlit dashboard]   
      ↓
[Airflow orquestração]  
      ↓
[Docker Compose tudo]   
      ↓
[Deploy na AWS/Azure]
```

Evolução natural do seu projeto `automacao-pesquisa-precos`!

---

## 📝 Notas Pessoais de Aprendizado
> (Adicione aqui insights que você vai acumulando nos estudos)

- 
- 
