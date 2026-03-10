# 📈 Metabase

> BI open source, simples e democrático. Ideal para times técnicos e startups.

**Relacionado:** [[PowerBI]] | [[Tableau]] | [[PostgreSQL]] | [[MySQL]] | [[MongoDB]] | [[Docker]]

---

## Instalação
```bash
# Docker (mais prático)
docker run -d \
  --name metabase \
  -p 3000:3000 \
  -e MB_DB_TYPE=postgres \
  -e MB_DB_DBNAME=metabase \
  -e MB_DB_PORT=5432 \
  -e MB_DB_USER=admin \
  -e MB_DB_PASS=senha \
  -e MB_DB_HOST=postgres \
  -v metabase_data:/metabase.db \
  metabase/metabase:latest

# Acessar: http://localhost:3000
# Setup inicial no browser
```

---

## Conceitos
- **Question:** consulta/visualização individual
- **Dashboard:** conjunto de questions agrupadas
- **Collection:** pasta para organizar questions e dashboards
- **Pulse:** relatório automático por email/Slack
- **Model:** query salva como tabela virtual (similar a uma view)

---

## SQL no Metabase
```sql
-- Filtros dinâmicos com variáveis Metabase
-- [[]] torna o filtro opcional
SELECT *
FROM pedidos
WHERE 1=1
  [[AND status = {{status}}]]           -- filtro de texto
  [[AND valor > {{valor_minimo}}]]      -- filtro numérico
  [[AND data >= {{data_inicio}}]]       -- filtro de data
  [[AND cidade IN ({{cidades}})]]       -- filtro de lista

-- Campo dinâmico
SELECT *
FROM tabela
WHERE {{filtro_campo}}                  -- field filter — Metabase reconhece tipos
```

---

## Snippets — Reutilizar SQL
```sql
-- Criar snippet "clientes_ativos"
SELECT id, nome, email FROM clientes WHERE ativo = true

-- Usar em outra query
SELECT p.*, c.nome as cliente_nome
FROM pedidos p
JOIN ({{snippet: clientes_ativos}}) c ON c.id = p.cliente_id
```

---

## API REST do Metabase
```python
import requests

BASE_URL = "http://localhost:3000/api"

# Autenticar
auth = requests.post(f"{BASE_URL}/session", json={
    "username": "admin@empresa.com",
    "password": "senha"
})
token = auth.json()["id"]
headers = {"X-Metabase-Session": token}

# Executar uma query
resposta = requests.post(
    f"{BASE_URL}/dataset",
    headers=headers,
    json={
        "database": 1,
        "type": "native",
        "native": {"query": "SELECT * FROM vendas LIMIT 100"}
    }
)
dados = resposta.json()["data"]["rows"]
```

---

## Metabase vs Power BI vs Tableau

| | Metabase | Power BI | Tableau |
|--|--|--|--|
| Licença | Open source | Pago | Pago |
| Curva de aprendizado | Baixa | Média | Alta |
| Self-service | ✅ | ✅ | ✅ |
| SQL nativo | ✅ Excelente | Limitado | Limitado |
| Visualizações | Básicas | Avançadas | Muito avançadas |
| Embed em apps | ✅ | Limitado | Limitado |
| Mobile | ✅ | ✅ | ✅ |
