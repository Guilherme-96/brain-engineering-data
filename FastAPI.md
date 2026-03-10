# ⚡ FastAPI

> Framework web moderno, rápido e com tipagem automática. O melhor Python para APIs de dados.

**Relacionado:** [[Python]] | [[REST_API]] | [[SQLAlchemy]] | [[PostgreSQL]] | [[Docker]] | [[Deploys]] | [[Streamlit]] | [[AWS]] | [[Azure]]

---

## Por que FastAPI?

```
Flask:   simples, mas sem tipagem, sem docs automáticas, sem async nativo
Django:  completo mas pesado para APIs simples
FastAPI: moderno, async, tipagem, OpenAPI automático, performático
```

**Instalação:**
```bash
pip install "fastapi[standard]"   # inclui uvicorn e httpx
```

---

## API Mínima
```python
from fastapi import FastAPI

app = FastAPI(title="API de Dados", version="1.0.0")

@app.get("/")
def health_check():
    return {"status": "ok"}

@app.get("/vendas/{id}")
def buscar_venda(id: int):
    return {"id": id, "valor": 150.00}

# Rodar:
# uvicorn main:app --reload --host 0.0.0.0 --port 8000
# Docs: http://localhost:8000/docs (Swagger automático!)
```

---

## Schemas com Pydantic — Validação Automática
```python
from pydantic import BaseModel, Field, validator, EmailStr
from typing import Optional
from datetime import datetime
from enum import Enum

class StatusPedido(str, Enum):
    PENDENTE = "pendente"
    APROVADO = "aprovado"
    CANCELADO = "cancelado"

class PedidoCreate(BaseModel):
    cliente_id: int = Field(..., gt=0, description="ID do cliente")
    valor: float = Field(..., gt=0, le=1_000_000, description="Valor em reais")
    status: StatusPedido = StatusPedido.PENDENTE
    observacao: Optional[str] = Field(None, max_length=500)

    @validator("valor")
    def validar_valor(cls, v):
        return round(v, 2)

class PedidoResponse(BaseModel):
    id: int
    cliente_id: int
    valor: float
    status: StatusPedido
    criado_em: datetime

    class Config:
        from_attributes = True    # permite criar de ORM SQLAlchemy

class PaginatedResponse(BaseModel):
    data: list[PedidoResponse]
    total: int
    page: int
    per_page: int
```

---

## Rotas Completas com CRUD
```python
from fastapi import FastAPI, HTTPException, Depends, Query, status
from sqlalchemy.orm import Session
from typing import Annotated

app = FastAPI()

# Dependency injection para sessão do banco
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

DbDep = Annotated[Session, Depends(get_db)]


@app.get("/pedidos", response_model=PaginatedResponse)
def listar_pedidos(
    db: DbDep,
    page: int = Query(1, ge=1),
    per_page: int = Query(20, ge=1, le=100),
    status: Optional[StatusPedido] = None,
    data_inicio: Optional[date] = None,
):
    query = db.query(Pedido)
    
    if status:
        query = query.filter(Pedido.status == status)
    if data_inicio:
        query = query.filter(Pedido.criado_em >= data_inicio)
    
    total = query.count()
    pedidos = query.offset((page - 1) * per_page).limit(per_page).all()
    
    return PaginatedResponse(data=pedidos, total=total, page=page, per_page=per_page)


@app.get("/pedidos/{id}", response_model=PedidoResponse)
def buscar_pedido(id: int, db: DbDep):
    pedido = db.query(Pedido).filter(Pedido.id == id).first()
    if not pedido:
        raise HTTPException(status_code=404, detail=f"Pedido {id} não encontrado")
    return pedido


@app.post("/pedidos", response_model=PedidoResponse, status_code=status.HTTP_201_CREATED)
def criar_pedido(dados: PedidoCreate, db: DbDep):
    # Verificar se cliente existe
    cliente = db.query(Cliente).filter(Cliente.id == dados.cliente_id).first()
    if not cliente:
        raise HTTPException(status_code=404, detail="Cliente não encontrado")
    
    pedido = Pedido(**dados.model_dump())
    db.add(pedido)
    db.commit()
    db.refresh(pedido)
    return pedido


@app.patch("/pedidos/{id}", response_model=PedidoResponse)
def atualizar_pedido(id: int, dados: PedidoUpdate, db: DbDep):
    pedido = db.query(Pedido).filter(Pedido.id == id).first()
    if not pedido:
        raise HTTPException(status_code=404, detail="Pedido não encontrado")
    
    # Só atualiza campos enviados
    for campo, valor in dados.model_dump(exclude_unset=True).items():
        setattr(pedido, campo, valor)
    
    db.commit()
    db.refresh(pedido)
    return pedido


@app.delete("/pedidos/{id}", status_code=status.HTTP_204_NO_CONTENT)
def deletar_pedido(id: int, db: DbDep):
    pedido = db.query(Pedido).filter(Pedido.id == id).first()
    if not pedido:
        raise HTTPException(status_code=404, detail="Pedido não encontrado")
    db.delete(pedido)
    db.commit()
```

---

## Autenticação JWT
```python
from fastapi import Depends, HTTPException, Security
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
import jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)], db: DbDep):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = int(payload["sub"])
    except (jwt.InvalidTokenError, KeyError):
        raise HTTPException(status_code=401, detail="Token inválido")
    
    user = db.query(Usuario).filter(Usuario.id == user_id).first()
    if not user:
        raise HTTPException(status_code=401, detail="Usuário não encontrado")
    return user

# Proteger rota
@app.get("/meus-pedidos", response_model=list[PedidoResponse])
def meus_pedidos(
    current_user: Annotated[Usuario, Depends(get_current_user)],
    db: DbDep
):
    return db.query(Pedido).filter(Pedido.cliente_id == current_user.id).all()
```

---

## Background Tasks e Async
```python
from fastapi import BackgroundTasks
import asyncio
import httpx

# Background task — executa após retornar resposta
def enviar_email_confirmacao(email: str, pedido_id: int):
    # lógica de envio de email (pode ser lenta)
    pass

@app.post("/pedidos", response_model=PedidoResponse)
def criar_pedido(dados: PedidoCreate, db: DbDep, bg: BackgroundTasks):
    pedido = Pedido(**dados.model_dump())
    db.add(pedido)
    db.commit()
    
    # Adiciona à fila de background (não bloqueia a resposta!)
    bg.add_task(enviar_email_confirmacao, dados.email, pedido.id)
    
    return pedido


# Endpoints assíncronos (I/O sem bloquear)
@app.get("/cotacao/{moeda}")
async def cotacao(moeda: str):
    async with httpx.AsyncClient() as client:
        resp = await client.get(f"https://api.exchangerate.host/latest?base={moeda}")
        return resp.json()
```

---

## Dockerfile para FastAPI
```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Usuário não-root
RUN useradd --no-create-home appuser
USER appuser

EXPOSE 8000

# Produção: múltiplos workers com gunicorn
CMD ["gunicorn", "main:app", \
     "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--workers", "4", \
     "--bind", "0.0.0.0:8000", \
     "--timeout", "120"]
```

---

## ✅ Quando usar FastAPI / ❌ Quando não usar

✅ USE FastAPI quando:
- Criar APIs de dados em Python
- Servir modelos de ML
- Microsserviços
- Precisa de docs automáticas (Swagger/OpenAPI)
- Time já sabe Python

❌ NÃO USE FastAPI quando:
- App web com templates HTML → use Django
- API muito simples (um endpoint) → Lambda basta
- Time tem mais expertise em Node.js → use Express/NestJS
