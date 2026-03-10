# 🔗 SQLAlchemy

> ORM e toolkit SQL para [[Python]]. Ponte entre código Python e bancos relacionais.

**Relacionado:** [[Python]] | [[Pandas]] | [[PostgreSQL]] | [[MySQL]] | [[SQLServer]] | [[Docker]]

**Instalação:**
```bash
pip install sqlalchemy
pip install psycopg2-binary    # PostgreSQL
pip install pymysql            # MySQL
pip install pyodbc             # SQL Server
```

---

## Strings de Conexão
```python
from sqlalchemy import create_engine

# PostgreSQL
engine = create_engine("postgresql+psycopg2://user:senha@localhost:5432/banco")

# MySQL
engine = create_engine("mysql+pymysql://user:senha@localhost:3306/banco")

# SQL Server
engine = create_engine("mssql+pyodbc://user:senha@servidor/banco?driver=ODBC+Driver+17+for+SQL+Server")

# SQLite (arquivo local)
engine = create_engine("sqlite:///local.db")
engine = create_engine("sqlite:///:memory:")    # em memória

# Com pool de conexões
engine = create_engine(
    "postgresql+psycopg2://user:senha@localhost/banco",
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True    # verifica conexão antes de usar
)
```

---

## SQL Core (baixo nível)
```python
from sqlalchemy import text

with engine.connect() as conn:
    # Query simples
    resultado = conn.execute(text("SELECT * FROM usuarios"))
    for linha in resultado:
        print(linha)
    
    # Com parâmetros (evita SQL injection)
    resultado = conn.execute(
        text("SELECT * FROM usuarios WHERE ativo = :ativo AND cidade = :cidade"),
        {"ativo": True, "cidade": "SP"}
    )
    
    # Fetch
    linha = resultado.fetchone()        # uma linha
    linhas = resultado.fetchall()       # todas
    df_dict = resultado.mappings().all() # lista de dicts

# Transação explícita
with engine.begin() as conn:
    conn.execute(text("UPDATE saldo SET valor = valor + 100 WHERE id = :id"), {"id": 1})
    conn.execute(text("INSERT INTO historico (descricao) VALUES (:desc)"), {"desc": "crédito"})
    # commit automático ao sair do with (ou rollback em exceção)
```

---

## Integração com Pandas
```python
import pandas as pd

# Ler do banco
df = pd.read_sql("SELECT * FROM vendas WHERE ano = 2024", con=engine)
df = pd.read_sql_table("clientes", con=engine)

# Escrever no banco
df.to_sql(
    name="vendas_processadas",
    con=engine,
    if_exists="replace",   # "replace", "append", "fail"
    index=False,
    chunksize=1000,        # insere em lotes
    dtype={                # tipos específicos
        "id": Integer(),
        "valor": Numeric(15, 2)
    }
)
```

---

## ORM — Mapeamento Objeto-Relacional
```python
from sqlalchemy.orm import DeclarativeBase, Session, relationship
from sqlalchemy import Column, Integer, String, Boolean, ForeignKey, DateTime
from datetime import datetime

class Base(DeclarativeBase):
    pass

class Usuario(Base):
    __tablename__ = "usuarios"
    
    id = Column(Integer, primary_key=True)
    nome = Column(String(100), nullable=False)
    email = Column(String(200), unique=True, nullable=False)
    ativo = Column(Boolean, default=True)
    criado_em = Column(DateTime, default=datetime.utcnow)
    
    pedidos = relationship("Pedido", back_populates="usuario")
    
    def __repr__(self):
        return f"<Usuario {self.nome}>"

class Pedido(Base):
    __tablename__ = "pedidos"
    
    id = Column(Integer, primary_key=True)
    usuario_id = Column(Integer, ForeignKey("usuarios.id"))
    valor = Column(Numeric(15, 2))
    
    usuario = relationship("Usuario", back_populates="pedidos")

# Criar tabelas
Base.metadata.create_all(engine)

# CRUD com Session
with Session(engine) as session:
    # Criar
    novo = Usuario(nome="João", email="joao@email.com")
    session.add(novo)
    session.commit()
    
    # Ler
    user = session.get(Usuario, 1)
    todos = session.query(Usuario).filter(Usuario.ativo == True).all()
    
    # Atualizar
    user.nome = "João Silva"
    session.commit()
    
    # Deletar
    session.delete(user)
    session.commit()
```
