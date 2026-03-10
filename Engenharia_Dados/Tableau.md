# 🎨 Tableau

> Ferramenta de visualização de dados líder de mercado. Poderosa para análises visuais complexas.

**Relacionado:** [[PowerBI]] | [[Metabase]] | [[PostgreSQL]] | [[Modelagem_de_Dados]] | [[Pandas]]

---

## Produtos Tableau
- **Tableau Desktop:** criação de visualizações (pago)
- **Tableau Public:** versão gratuita (dados públicos)
- **Tableau Server:** compartilhamento on-premise
- **Tableau Cloud:** compartilhamento na nuvem
- **Tableau Prep:** preparação e limpeza de dados

---

## Conectores
- Banco de dados: PostgreSQL, MySQL, SQL Server, Redshift, BigQuery, Snowflake
- Arquivos: Excel, CSV, JSON, Parquet, Spatial
- Cloud: AWS, Azure, Google Cloud
- Web: Google Analytics, Salesforce
- Python/R: TabPy (cálculos avançados)

---

## Conceitos Fundamentais

### Tipos de Campos
- **Dimensão:** campos categóricos/textuais (azul no Tableau)
- **Medida:** campos numéricos que podem ser agregados (verde)
- **Dimensão de Data:** datas (hierarquia automática: ano > trimestre > mês > dia)
- **Calculado:** campo criado por fórmula

### Tipos de Conexão
- **Live:** consulta o banco a cada interação (dados atualizados)
- **Extract:** cópia local dos dados (mais rápido, atualização agendada)

---

## Cálculos (Tableau Formulas)
```
// Operadores básicos
[Receita] - [Custo]
[Vendas] / [Meta] * 100

// Condicionais
IF [Valor] > 1000 THEN "Alto"
ELSEIF [Valor] > 100 THEN "Médio"
ELSE "Baixo"
END

// CASE
CASE [Status]
    WHEN "A" THEN "Aprovado"
    WHEN "P" THEN "Pendente"
    ELSE "Cancelado"
END

// Date functions
DATETRUNC('month', [Data])
DATEDIFF('day', [Data Inicio], [Data Fim])
DATEADD('month', -1, TODAY())
YEAR([Data])
MONTH([Data])

// String functions
UPPER([Nome])
TRIM([Email])
LEFT([CPF], 3)
CONTAINS([Descricao], "urgente")
REGEXP_MATCH([Texto], "^\d{5}-?\d{3}$")

// Aggregation
SUM([Valor])
AVG([Valor])
COUNTD([Cliente ID])          // count distinct
MEDIAN([Valor])
PERCENTILE([Valor], 0.95)
WINDOW_SUM(SUM([Valor]))      // soma em janela deslizante

// LOD — Level of Detail (poder único do Tableau)
// Fixado em dimensão independente da view
{FIXED [Cliente ID] : MAX([Data Compra])}  // última compra por cliente
{FIXED [Região] : SUM([Vendas])}           // total por região em qualquer view
{INCLUDE [Produto] : SUM([Vendas])}        // inclui produto no cálculo
{EXCLUDE [Mês] : SUM([Vendas])}           // ignora mês no cálculo
```

---

## LOD Expressions — Casos de Uso
```
// Coorte — quando o cliente fez a primeira compra?
{FIXED [Cliente ID] : MIN([Data])}

// Ticket médio global (independente de filtros da view)
{FIXED : AVG([Valor])}

// Rank de produto dentro de cada categoria
RANK(SUM([Vendas]))  // + Compute Using: Produto

// % do total
SUM([Vendas]) / {FIXED : SUM([Vendas])} * 100
```

---

## Tableau com Python (TabPy)
```python
# Instalar TabPy
pip install tabpy
tabpy &   # subir servidor na porta 9004

# No Tableau Desktop:
# Help → Settings → External Services → TabPy

# Fórmula no Tableau
SCRIPT_REAL("
import numpy as np
return list(np.percentile(_arg1, [50]))
", SUM([Valor]))

# Ou scripts mais complexos
SCRIPT_STR("
import pandas as pd
df = pd.DataFrame({'valor': _arg1, 'categoria': _arg2})
result = df.groupby('categoria')['valor'].transform('rank')
return list(result.astype(str))
", ATTR([Valor]), ATTR([Categoria]))
```

---

## Dicas de Performance
- Use **Extracts** em vez de Live para dados grandes
- **Aggregate to visible dimensions** nas propriedades de extract
- Reduza granularidade de datas (mês/trimestre em vez de dia)
- Use filtros de contexto para pre-filtrar dados
- Evite FIXED LOD excessivos — são custosos
- Otimize queries com custom SQL para JOINs complexos
