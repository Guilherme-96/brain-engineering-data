# 🐼 Pandas

> Biblioteca principal para manipulação de dados tabulares em [[Python]].

**Relacionado:** [[Python]] | [[NumPy]] | [[SQLAlchemy]] | [[Spark_PySpark]] | [[PostgreSQL]] | [[AWS]]

**Instalação:** `pip install pandas`
**Docs:** https://pandas.pydata.org/docs/

---

## Leitura de Dados
```python
import pandas as pd

df = pd.read_csv("dados.csv", sep=",", encoding="utf-8")
df = pd.read_excel("planilha.xlsx", sheet_name="Sheet1")
df = pd.read_json("dados.json")
df = pd.read_parquet("dados.parquet")       # formato colunar, eficiente
df = pd.read_sql("SELECT * FROM tabela", con=engine)
df = pd.read_sql_table("tabela", con=engine)

# CSV com opções avançadas
df = pd.read_csv(
    "dados.csv",
    sep=";",
    encoding="latin-1",
    parse_dates=["data_venda"],
    dtype={"cpf": str, "cep": str},
    usecols=["id", "nome", "valor"],
    nrows=1000,                             # ler só as primeiras 1000 linhas
    skiprows=2                              # pular linhas do início
)
```

---

## Inspeção do DataFrame
```python
df.head(10)           # primeiras 10 linhas
df.tail(5)            # últimas 5 linhas
df.info()             # tipos e nulos
df.describe()         # estatísticas descritivas
df.shape              # (linhas, colunas)
df.columns.tolist()   # lista de colunas
df.dtypes             # tipo de cada coluna
df.isnull().sum()     # contagem de nulos por coluna
df.nunique()          # valores únicos por coluna
df.value_counts("coluna")  # frequência de valores
```

---

## Seleção e Filtro
```python
df["coluna"]                           # série
df[["col1", "col2"]]                   # dataframe
df.loc[0:5, ["col1", "col2"]]         # por label
df.iloc[0:5, 0:2]                     # por posição
df[df["idade"] > 18]                  # filtro booleano
df.query("idade > 18 and cidade == 'SP'")
df.loc[df["valor"].between(100, 500)]
df[df["nome"].str.contains("Silva", na=False)]
```

---

## Limpeza de Dados
```python
# Nulos
df = df.dropna(subset=["coluna_importante"])
df = df.fillna({"col1": 0, "col2": "desconhecido"})
df["col"] = df["col"].fillna(df["col"].median())

# Duplicatas
df = df.drop_duplicates(subset=["id"])
df = df.drop_duplicates(keep="last")

# Tipos
df["data"] = pd.to_datetime(df["data"], format="%Y-%m-%d")
df["valor"] = pd.to_numeric(df["valor"], errors="coerce")
df["categoria"] = df["categoria"].astype("category")

# Strings
df["nome"] = df["nome"].str.strip().str.lower()
df["cpf"] = df["cpf"].str.replace(r"\D", "", regex=True)

# Renomear colunas
df.columns = df.columns.str.lower().str.replace(" ", "_")
df = df.rename(columns={"old_name": "new_name"})
```

---

## Transformações
```python
# Novas colunas
df["lucro"] = df["receita"] - df["custo"]
df["desconto_pct"] = df["desconto"] / df["preco"] * 100

# Apply
df["categoria"] = df["valor"].apply(
    lambda x: "alto" if x > 1000 else "baixo"
)

# Map (para séries)
mapa = {"SP": "São Paulo", "RJ": "Rio de Janeiro"}
df["estado_nome"] = df["estado"].map(mapa)

# Where / Mask
df["ajustado"] = df["valor"].where(df["valor"] > 0, other=0)

# Extração de datas
df["ano"] = df["data"].dt.year
df["mes"] = df["data"].dt.month
df["dia_semana"] = df["data"].dt.day_name()
```

---

## Agrupamento
```python
resumo = (df
    .groupby("categoria")
    .agg(
        total=("valor", "sum"),
        media=("valor", "mean"),
        mediana=("valor", "median"),
        contagem=("id", "count"),
        maximo=("valor", "max")
    )
    .reset_index()
    .sort_values("total", ascending=False)
)

# Múltiplas colunas de agrupamento
df.groupby(["ano", "mes", "categoria"])["valor"].sum()
```

---

## Merge / Join
```python
# Equivalente ao SQL JOIN
resultado = pd.merge(df_esq, df_dir, on="id", how="left")
# how: "left", "right", "inner", "outer"

# Múltiplas chaves
pd.merge(df1, df2, on=["id", "data"], how="inner")

# Chaves com nomes diferentes
pd.merge(df1, df2, left_on="user_id", right_on="id")

# Concatenar linhas
pd.concat([df1, df2, df3], ignore_index=True)
```

---

## Pivot e Reshape
```python
# Pivot table
tabela = df.pivot_table(
    values="vendas",
    index="mes",
    columns="categoria",
    aggfunc="sum",
    fill_value=0
)

# Melt (de wide para long)
df_long = df.melt(
    id_vars=["id", "nome"],
    value_vars=["jan", "fev", "mar"],
    var_name="mes",
    value_name="valor"
)
```

---

## Escrita
```python
df.to_csv("saida.csv", index=False, encoding="utf-8")
df.to_parquet("saida.parquet", index=False)
df.to_excel("saida.xlsx", index=False, sheet_name="Dados")
df.to_sql("tabela", con=engine, if_exists="replace", index=False)
df.to_json("saida.json", orient="records")
```

---

## Performance com Dados Grandes
```python
# Ler em chunks
for chunk in pd.read_csv("gigante.csv", chunksize=50_000):
    processar(chunk)

# Reduzir memória com tipos menores
df["int_col"] = df["int_col"].astype("int32")   # em vez de int64
df["float_col"] = df["float_col"].astype("float32")

# Usar categorias para strings repetidas
df["uf"] = df["uf"].astype("category")

# Parquet é muito mais eficiente que CSV para grandes volumes
df.to_parquet("dados.parquet", compression="snappy")
```
