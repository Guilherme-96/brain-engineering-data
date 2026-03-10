# 📊 OpenPyXL

> Leitura e escrita de arquivos `.xlsx` sem precisar do Excel instalado. Puro [[Python]].

**Relacionado:** [[Python]] | [[Pandas]] | [[Windows]]

**Instalação:** `pip install openpyxl`

---

## Criar Planilha
```python
import openpyxl
from openpyxl import Workbook

wb = Workbook()
ws = wb.active
ws.title = "Dados"

# Escrever células
ws["A1"] = "Nome"
ws["B1"] = "Valor"
ws["C1"] = "Data"

# Escrever por linha
ws.append(["João", 1500.00, "2024-01-15"])
ws.append(["Maria", 2000.00, "2024-01-16"])

# Linhas e colunas
ws.cell(row=4, column=1, value="Carlos")
ws.cell(row=4, column=2, value=1800)

wb.save("relatorio.xlsx")
```

---

## Ler Planilha Existente
```python
wb = openpyxl.load_workbook("relatorio.xlsx")

# Listar abas
print(wb.sheetnames)  # ['Dados', 'Resumo']

# Acessar aba
ws = wb["Dados"]
ws = wb.active         # aba ativa

# Ler células
valor = ws["A1"].value
valor = ws.cell(row=1, column=1).value

# Iterar linhas
for linha in ws.iter_rows(min_row=2, values_only=True):
    print(linha)       # (valor1, valor2, valor3)

# Iterar com cabeçalho como dicionário
headers = [ws.cell(1, col).value for col in range(1, ws.max_column+1)]
for linha in ws.iter_rows(min_row=2, values_only=True):
    registro = dict(zip(headers, linha))
    print(registro)
```

---

## Estilização
```python
from openpyxl.styles import (
    Font, PatternFill, Alignment, Border, Side, numbers
)

ws = wb.active

# Fonte
ws["A1"].font = Font(
    bold=True,
    color="FFFFFF",
    size=12,
    name="Calibri"
)

# Cor de fundo
ws["A1"].fill = PatternFill(
    fill_type="solid",
    fgColor="366092"   # azul escuro
)

# Alinhamento
ws["A1"].alignment = Alignment(
    horizontal="center",
    vertical="center",
    wrap_text=True
)

# Borda
borda = Border(
    bottom=Side(style="thin", color="000000")
)
ws["A1"].border = borda

# Formato de número
ws["B2"].number_format = numbers.FORMAT_CURRENCY_USD_SIMPLE
ws["C2"].number_format = "DD/MM/YYYY"

# Largura de coluna
ws.column_dimensions["A"].width = 20
ws.column_dimensions["B"].width = 15

# Altura de linha
ws.row_dimensions[1].height = 25

# Mesclar células
ws.merge_cells("A1:C1")
```

---

## Fórmulas e Gráficos
```python
# Fórmulas (escreve a fórmula, Excel calcula)
ws["D2"] = "=B2*1.1"
ws["D10"] = "=SUM(D2:D9)"
ws["D11"] = "=AVERAGE(D2:D9)"

# Gráfico de barras
from openpyxl.chart import BarChart, Reference

chart = BarChart()
chart.title = "Vendas por Mês"
chart.y_axis.title = "Valor"
chart.x_axis.title = "Mês"

data = Reference(ws, min_col=2, min_row=1, max_row=ws.max_row)
cats = Reference(ws, min_col=1, min_row=2, max_row=ws.max_row)

chart.add_data(data, titles_from_data=True)
chart.set_categories(cats)
chart.width = 15
chart.height = 10

ws.add_chart(chart, "E2")
wb.save("relatorio.xlsx")
```

---

## Integração com Pandas
```python
import pandas as pd

# Pandas pode ler/escrever xlsx usando openpyxl como engine
df = pd.read_excel("planilha.xlsx", sheet_name="Dados", engine="openpyxl")
df.to_excel("saida.xlsx", index=False, engine="openpyxl")

# Múltiplas abas
with pd.ExcelWriter("multi_abas.xlsx", engine="openpyxl") as writer:
    df_vendas.to_excel(writer, sheet_name="Vendas", index=False)
    df_estoque.to_excel(writer, sheet_name="Estoque", index=False)
    df_clientes.to_excel(writer, sheet_name="Clientes", index=False)
```
