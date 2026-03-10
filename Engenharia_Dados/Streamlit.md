# 📊 Streamlit

> Transforme scripts Python em aplicações web interativas em minutos. O jeito mais rápido de visualizar dados.

**Relacionado:** [[Python]] | [[Pandas]] | [[Plotly]] | [[FastAPI]] | [[PostgreSQL]] | [[SQLAlchemy]] | [[Deploys]] | [[AWS]] | [[Docker]]

---

## O que é Streamlit?

```python
# Isso já é um app web funcional:
import streamlit as st
import pandas as pd

df = pd.read_csv("vendas.csv")
st.title("Dashboard de Vendas")
st.dataframe(df)
st.line_chart(df.set_index("data")["valor"])

# rodar: streamlit run app.py
```

---

## Instalação e Execução
```bash
pip install streamlit plotly pandas sqlalchemy

streamlit run app.py
streamlit run app.py --server.port 8080 --server.headless true
```

---

## Componentes Essenciais

### Texto e Layout
```python
import streamlit as st

st.title("Título Grande")
st.header("Cabeçalho")
st.subheader("Sub-cabeçalho")
st.text("Texto simples")
st.markdown("**Negrito** e *itálico* e `código`")
st.code("print('hello')", language="python")
st.latex(r"E = mc^2")
st.divider()

# Colunas
col1, col2, col3 = st.columns([2, 1, 1])  # proporções
with col1:
    st.metric("Total Vendas", "R$ 1.2M", delta="+15%")
with col2:
    st.metric("Pedidos", "3.420", delta="+8%")
with col3:
    st.metric("Ticket Médio", "R$ 351", delta="-2%")

# Tabs
tab1, tab2, tab3 = st.tabs(["Visão Geral", "Detalhes", "Configurações"])
with tab1:
    st.write("Conteúdo da aba 1")

# Expander (acordeão)
with st.expander("Ver dados brutos"):
    st.dataframe(df)

# Sidebar
with st.sidebar:
    st.header("Filtros")
    data_inicio = st.date_input("Data início")
```

---

### Widgets Interativos
```python
# Inputs
nome = st.text_input("Nome", placeholder="Digite seu nome")
valor = st.number_input("Valor", min_value=0.0, step=0.01)
data = st.date_input("Data")
hora = st.time_input("Hora")

# Seleção
opcao = st.selectbox("Estado", ["SP", "RJ", "MG", "RS"])
multiplo = st.multiselect("Categorias", ["A", "B", "C"], default=["A"])
slider_val = st.slider("Meses anteriores", 1, 24, 6)
range_val = st.slider("Faixa de valor", 0, 10000, (100, 5000))

# Botões
if st.button("Processar"):
    with st.spinner("Processando..."):
        resultado = processar_dados()
    st.success("Concluído!")

clicado = st.button("Confirmar", type="primary")  # botão azul

# Upload
arquivo = st.file_uploader("Enviar CSV", type=["csv", "xlsx"])
if arquivo:
    df = pd.read_csv(arquivo)
    st.dataframe(df)

# Toggle e Checkbox
modo_escuro = st.toggle("Modo escuro")
aceito = st.checkbox("Aceito os termos")

# Radio
opcao = st.radio("Granularidade", ["Diário", "Mensal", "Anual"])
```

---

### Gráficos
```python
import plotly.express as px
import plotly.graph_objects as go

# Plotly (melhor para gráficos interativos)
fig = px.line(df, x="data", y="valor", color="categoria",
              title="Evolução de Vendas",
              labels={"valor": "R$", "data": "Data"})
fig.update_layout(hovermode="x unified")
st.plotly_chart(fig, use_container_width=True)

# Barras
fig = px.bar(df_resumo, x="categoria", y="total", 
             color="status", barmode="group")
st.plotly_chart(fig)

# Mapa
fig = px.choropleth(df_estados, locations="uf",
                    color="vendas", scope="south america")
st.plotly_chart(fig)

# Nativo do Streamlit (mais simples)
st.line_chart(df.set_index("data")["valor"])
st.bar_chart(df.groupby("categoria")["valor"].sum())
st.area_chart(df)
```

---

### Cache — Performance Essencial
```python
# @st.cache_data: para DataFrames e dados (reexecuta quando args mudam)
@st.cache_data(ttl=3600)  # expira em 1 hora
def carregar_vendas(data_inicio: str, data_fim: str) -> pd.DataFrame:
    return pd.read_sql(
        f"SELECT * FROM vendas WHERE data BETWEEN '{data_inicio}' AND '{data_fim}'",
        engine
    )

# @st.cache_resource: para objetos pesados (conexões, modelos ML)
@st.cache_resource
def get_engine():
    return create_engine("postgresql://...")

@st.cache_resource
def carregar_modelo():
    import joblib
    return joblib.load("modelo.pkl")

engine = get_engine()
modelo = carregar_modelo()
```

---

## App de Dashboard Completo
```python
# dashboard.py
import streamlit as st
import pandas as pd
import plotly.express as px
from sqlalchemy import create_engine

st.set_page_config(
    page_title="Dashboard de Vendas",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

@st.cache_resource
def get_engine():
    return create_engine("postgresql://admin:senha@localhost/pipeline")

@st.cache_data(ttl=300, show_spinner="Carregando dados...")
def carregar_dados(data_inicio, data_fim, estados):
    filtro_estados = f"AND uf IN ({','.join(repr(e) for e in estados)})" if estados else ""
    return pd.read_sql(f"""
        SELECT data, categoria, uf, SUM(valor) as total
        FROM fato_vendas
        WHERE data BETWEEN '{data_inicio}' AND '{data_fim}'
        {filtro_estados}
        GROUP BY 1, 2, 3
    """, get_engine())

# Sidebar
with st.sidebar:
    st.image("logo.png", width=150)
    st.header("🔍 Filtros")
    
    col1, col2 = st.columns(2)
    data_inicio = col1.date_input("De")
    data_fim = col2.date_input("Até")
    
    estados = st.multiselect("Estados", ["SP","RJ","MG","RS","PR","SC","BA"])
    
    if st.button("🔄 Atualizar", type="primary"):
        st.cache_data.clear()

# Carregar dados
df = carregar_dados(data_inicio, data_fim, estados)

if df.empty:
    st.warning("Nenhum dado encontrado para os filtros selecionados.")
    st.stop()

# KPIs
total = df.total.sum()
col1, col2, col3, col4 = st.columns(4)
col1.metric("💰 Total", f"R$ {total/1e6:.1f}M")
col2.metric("📦 Categorias", df.categoria.nunique())
col3.metric("🗺️ Estados", df.uf.nunique())
col4.metric("📅 Dias", df.data.nunique())

# Gráficos
col1, col2 = st.columns(2)
with col1:
    fig = px.line(df.groupby("data")["total"].sum().reset_index(),
                  x="data", y="total", title="Evolução Temporal")
    st.plotly_chart(fig, use_container_width=True)

with col2:
    fig = px.pie(df.groupby("categoria")["total"].sum().reset_index(),
                 values="total", names="categoria", title="Por Categoria")
    st.plotly_chart(fig, use_container_width=True)
```

---

## ✅ Quando usar Streamlit / ❌ Quando não usar

✅ USE quando:
- Protótipo de dashboard rápido (horas)
- Ferramentas internas para time de dados
- Aplicação de exploração de dados
- Demonstrar modelos de ML
- Substituto simples para Metabase/Power BI

❌ NÃO USE quando:
- Muitos usuários simultâneos (use Power BI ou Metabase)
- Precisa de auth complexa e multi-tenant
- App precisa de performance de produção → use FastAPI + React
- Usuários não-técnicos que precisam criar próprios relatórios
