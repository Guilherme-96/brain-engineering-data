# 🌐 REST API — Fundamentos

> Padrão de comunicação entre sistemas via HTTP. A língua franca da internet.

**Relacionado:** [[FastAPI]] | [[Python]] | [[Requests]] | [[Docker]] | [[AWS]] | [[Azure]] | [[Event_Driven_SQS_Lambda]]

---

## O que é REST?

**REST** = Representational State Transfer

Princípios:
- **Stateless:** cada requisição é independente (servidor não guarda estado do cliente)
- **Uniform Interface:** verbos HTTP têm significado definido
- **Resource-based:** URLs representam recursos (substantivos, não verbos)

---

## Verbos HTTP e seus Significados

| Verbo | Ação | Idempotente? | Body? |
|-------|------|:---:|:---:|
| GET | Ler / Buscar | ✅ | ❌ |
| POST | Criar | ❌ | ✅ |
| PUT | Substituir completo | ✅ | ✅ |
| PATCH | Atualizar parcial | ✅ | ✅ |
| DELETE | Remover | ✅ | ❌ |

---

## URLs RESTful — Boas Práticas

```
# ✅ BOM — substantivos, hierarquia clara
GET    /usuarios                    # listar todos
GET    /usuarios/123                # buscar um
POST   /usuarios                    # criar
PUT    /usuarios/123                # substituir completo
PATCH  /usuarios/123                # atualizar parcial
DELETE /usuarios/123                # remover

GET    /usuarios/123/pedidos        # pedidos do usuário 123
GET    /usuarios/123/pedidos/456    # pedido específico

# ❌ RUIM — verbos na URL
GET    /buscarUsuario?id=123
POST   /criarUsuario
GET    /deletarPedido?id=456
```

---

## Códigos de Status HTTP

```
2xx — Sucesso
200 OK             → GET/PUT/PATCH com sucesso
201 Created        → POST criou um recurso
204 No Content     → DELETE com sucesso (sem body)

3xx — Redirecionamento
301 Moved Permanently
302 Found (temporário)

4xx — Erro do cliente
400 Bad Request    → body inválido, parâmetro errado
401 Unauthorized   → não autenticado (sem token)
403 Forbidden      → autenticado mas sem permissão
404 Not Found      → recurso não existe
409 Conflict       → conflito (ex: email duplicado)
422 Unprocessable  → validação falhou
429 Too Many Requests → rate limit atingido

5xx — Erro do servidor
500 Internal Server Error → bug no seu código
502 Bad Gateway           → serviço upstream caiu
503 Service Unavailable   → servidor sobrecarregado
```

---

## Autenticação

### JWT (JSON Web Token)
```python
import jwt
from datetime import datetime, timedelta

SECRET = "minha-chave-secreta-muito-longa"

def criar_token(user_id: int, email: str) -> str:
    payload = {
        "sub": str(user_id),        # subject
        "email": email,
        "iat": datetime.utcnow(),   # issued at
        "exp": datetime.utcnow() + timedelta(hours=8)  # expira em 8h
    }
    return jwt.encode(payload, SECRET, algorithm="HS256")

def verificar_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET, algorithms=["HS256"])
    except jwt.ExpiredSignatureError:
        raise ValueError("Token expirado")
    except jwt.InvalidTokenError:
        raise ValueError("Token inválido")

# Header HTTP:
# Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key
```python
# Simples e direto para APIs de dados
# Header: X-API-Key: minha-chave-secreta

# Ou query param (menos seguro):
# GET /dados?api_key=minha-chave
```

---

## Versionamento de API

```
# URL versioning (mais simples)
/api/v1/usuarios
/api/v2/usuarios     # nova versão, v1 continua funcionando

# Header versioning (mais limpo)
Accept: application/vnd.minha-api.v2+json

# Regra de ouro: NUNCA quebre contratos sem versionar
# Adicionar campos é retrocompatível ✅
# Remover ou renomear campos quebra clientes ❌
```

---

## Paginação

```
# Offset (simples)
GET /pedidos?page=2&per_page=50

# Cursor (melhor para performance em tabelas grandes)
GET /pedidos?cursor=eyJpZCI6MTAwfQ&limit=50

# Resposta padronizada
{
  "data": [...],
  "pagination": {
    "total": 1250,
    "page": 2,
    "per_page": 50,
    "total_pages": 25,
    "next": "/pedidos?page=3&per_page=50",
    "prev": "/pedidos?page=1&per_page=50"
  }
}
```

---

## Documentação — OpenAPI / Swagger

```yaml
# openapi.yml
openapi: "3.0.3"
info:
  title: "API de Dados de Vendas"
  version: "1.0.0"
paths:
  /vendas:
    get:
      summary: "Listar vendas"
      parameters:
        - name: data_inicio
          in: query
          schema: {type: string, format: date}
        - name: status
          in: query
          schema: {type: string, enum: [aprovado, pendente, cancelado]}
      responses:
        "200":
          description: "Lista de vendas"
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Venda"
```

*Ver [[FastAPI]] — gera OpenAPI automaticamente!*
