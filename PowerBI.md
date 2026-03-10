# 📊 Power BI

> Ferramenta de Business Intelligence da Microsoft. Dominante em empresas brasileiras.

**Relacionado:** [[Metabase]] | [[Tableau]] | [[PostgreSQL]] | [[MySQL]] | [[SQLServer]] | [[Modelagem_de_Dados]]

---

## Componentes Principais
- **Power Query (M):** transformação e limpeza de dados
- **DAX:** linguagem para medidas e colunas calculadas
- **Data Model:** relacionamentos entre tabelas
- **Visualizações:** gráficos, tabelas, mapas
- **Power BI Service:** publicação e compartilhamento na web

---

## Fontes de Dados
- SQL Server, PostgreSQL, MySQL, Oracle
- Excel, CSV, JSON, Parquet
- APIs REST (via Web Connector ou Power Query)
- Azure, AWS (S3, Redshift)
- Salesforce, Google Analytics, SharePoint

---

## Power Query (M Language)
```m
// Transformações de dados
let
    // 1. Conectar
    Fonte = Csv.Document(File.Contents("C:\dados.csv"), [Delimiter=",", Encoding=1252]),
    
    // 2. Promover headers
    Headers = Table.PromoteHeaders(Fonte),
    
    // 3. Tipos corretos
    Tipos = Table.TransformColumnTypes(Headers, {
        {"data", type date}, 
        {"valor", type number}, 
        {"nome", type text}
    }),
    
    // 4. Filtrar
    Filtrado = Table.SelectRows(Tipos, each [ativo] = true),
    
    // 5. Adicionar coluna
    ComColuna = Table.AddColumn(Filtrado, "mes", each Date.Month([data]), Int64.Type),
    
    // 6. Remover nulos
    SemNulos = Table.SelectRows(ComColuna, each [valor] <> null)
in
    SemNulos
```

---

## DAX — Data Analysis Expressions
```dax
// Medidas básicas
Total Vendas = SUM(Vendas[Valor])
Qtd Pedidos = COUNTROWS(Pedidos)
Ticket Médio = DIVIDE([Total Vendas], [Qtd Pedidos])

// Medida com filtro
Vendas SP = CALCULATE(
    [Total Vendas],
    Vendas[Estado] = "SP"
)

Vendas Periodo = CALCULATE(
    [Total Vendas],
    DATESBETWEEN(Calendario[Data], DATE(2024,1,1), DATE(2024,12,31))
)

// Inteligência temporal
Vendas Mês Anterior = CALCULATE(
    [Total Vendas],
    DATEADD(Calendario[Data], -1, MONTH)
)

YTD Vendas = TOTALYTD([Total Vendas], Calendario[Data])
YOY % = DIVIDE([Total Vendas] - [Vendas Ano Anterior], [Vendas Ano Anterior])

// Variáveis (mais legível)
Crescimento % =
VAR VendasAtual = [Total Vendas]
VAR VendasAnterior = [Vendas Mês Anterior]
RETURN
    IF(
        VendasAnterior <> 0,
        DIVIDE(VendasAtual - VendasAnterior, VendasAnterior),
        BLANK()
    )

// Ranking
Rank Produto = RANKX(ALL(Produtos[Nome]), [Total Vendas])

// Colunas calculadas (diferente de medidas!)
Categoria Valor = 
    IF(Vendas[Valor] > 1000, "Alto", "Baixo")
```

---

## Modelagem no Power BI
```
Estrela (Star Schema) — RECOMENDADO:
   dim_Produto
        |
dim_Tempo──fato_Vendas──dim_Cliente
        |
     dim_Loja

Boas práticas:
- Uma fato, várias dimensões
- Relacionamentos 1:N (da dimensão para fato)
- Tabela Calendário própria (melhor performance)
- Ocultar chaves FK das dimensões
```

---

## Tabela Calendário em DAX
```dax
Calendario =
ADDCOLUMNS(
    CALENDAR(DATE(2020,1,1), DATE(2030,12,31)),
    "Ano", YEAR([Date]),
    "Mês Num", MONTH([Date]),
    "Mês Nome", FORMAT([Date], "MMMM"),
    "Mês Curto", FORMAT([Date], "MMM"),
    "Trimestre", "T" & QUARTER([Date]),
    "Semana", WEEKNUM([Date]),
    "Dia Semana", WEEKDAY([Date]),
    "É Final de Semana", IF(WEEKDAY([Date], 2) >= 6, TRUE, FALSE),
    "Ano-Mês", FORMAT([Date], "YYYY-MM")
)
```
