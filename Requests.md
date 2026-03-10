# 🌐 Requests

> Biblioteca para fazer requisições HTTP em [[Python]]. A mais popular para consumir APIs.

**Relacionado:** [[Python]] | [[Pandas]] | [[Playwright]] | [[AWS]]

**Instalação:** `pip install requests`

---

## Métodos HTTP
```python
import requests

# GET — buscar dados
r = requests.get("https://api.exemplo.com/usuarios")

# POST — enviar dados
r = requests.post("https://api.exemplo.com/usuarios", json={"nome": "João"})

# PUT — atualizar completamente
r = requests.put("https://api.exemplo.com/usuarios/1", json={"nome": "João"})

# PATCH — atualizar parcialmente
r = requests.patch("https://api.exemplo.com/usuarios/1", json={"ativo": True})

# DELETE — remover
r = requests.delete("https://api.exemplo.com/usuarios/1")
```

---

## Parâmetros e Headers
```python
# Query params (?pagina=1&limite=100)
r = requests.get(
    "https://api.exemplo.com/dados",
    params={"pagina": 1, "limite": 100, "status": "ativo"}
)

# Headers (autenticação, content-type, etc.)
r = requests.get(
    "https://api.exemplo.com/dados",
    headers={
        "Authorization": "Bearer meu-token",
        "Accept": "application/json",
        "X-API-Key": "minha-api-key"
    },
    timeout=30        # tempo máximo de espera em segundos
)

# Autenticação básica
r = requests.get(url, auth=("usuario", "senha"))
```

---

## Resposta
```python
r = requests.get("https://api.exemplo.com/dados")

r.status_code          # 200, 404, 500, etc.
r.ok                   # True se status < 400
r.json()               # parse JSON → dict/list
r.text                 # resposta como string
r.content              # resposta como bytes
r.headers              # headers da resposta
r.url                  # URL final (após redirects)

# Lança exceção se status >= 400
r.raise_for_status()
```

---

## Tratamento de Erros
```python
from requests.exceptions import (
    HTTPError, ConnectionError, Timeout, RequestException
)

try:
    r = requests.get(url, timeout=10)
    r.raise_for_status()
    dados = r.json()
except HTTPError as e:
    print(f"Erro HTTP {e.response.status_code}: {e}")
except ConnectionError:
    print("Sem conexão com a internet")
except Timeout:
    print("Requisição demorou demais")
except RequestException as e:
    print(f"Erro genérico: {e}")
```

---

## Session — Reutilizar Conexão
```python
# Session reutiliza conexão TCP e mantém cookies/headers
with requests.Session() as session:
    session.headers.update({
        "Authorization": "Bearer TOKEN",
        "User-Agent": "MeuApp/1.0"
    })
    
    r1 = session.get("https://api.exemplo.com/endpoint1")
    r2 = session.get("https://api.exemplo.com/endpoint2")
    r3 = session.post("https://api.exemplo.com/dados", json=dados)
```

---

## Paginação de API
```python
def buscar_todos(url_base: str, token: str) -> list:
    """Busca todos os registros paginados de uma API."""
    todos = []
    pagina = 1
    
    with requests.Session() as s:
        s.headers["Authorization"] = f"Bearer {token}"
        
        while True:
            r = s.get(url_base, params={"page": pagina, "per_page": 100})
            r.raise_for_status()
            dados = r.json()
            
            if not dados.get("results"):
                break
            
            todos.extend(dados["results"])
            
            if not dados.get("next"):
                break
            
            pagina += 1
    
    return todos
```

---

## Retry com Backoff
```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry_strategy = Retry(
    total=3,                          # tentativas máximas
    backoff_factor=1,                 # espera 1, 2, 4 segundos
    status_forcelist=[429, 500, 502, 503, 504]
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session = requests.Session()
session.mount("https://", adapter)
session.mount("http://", adapter)
```
