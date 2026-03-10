# 🍃 MongoDB

> Banco de dados NoSQL orientado a documentos. Ideal para dados semi-estruturados e schemas flexíveis.

**Relacionado:** [[Modelagem_de_Dados]] | [[PostgreSQL]] | [[Docker]] | [[Python]] | [[AWS]]

---

## Conceitos Fundamentais
| MongoDB | SQL Equivalente |
|---------|----------------|
| Database | Database |
| Collection | Tabela |
| Document | Linha/Registro |
| Field | Coluna |
| `_id` | Chave Primária |
| Embedded document | JOIN (mas inline) |

---

## Instalação
```bash
# Docker (recomendado para dev)
docker run -d \
  --name mongo \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=senha123 \
  -p 27017:27017 \
  -v mongo_data:/data/db \
  mongo:7

# Conectar via mongosh
docker exec -it mongo mongosh -u root -p senha123
```

---

## Operações com PyMongo
```python
from pymongo import MongoClient, ASCENDING, DESCENDING
from datetime import datetime

client = MongoClient("mongodb://root:senha123@localhost:27017/")
db = client["meu_banco"]
colecao = db["usuarios"]

# Inserir
colecao.insert_one({
    "nome": "João",
    "email": "joao@email.com",
    "idade": 30,
    "endereco": {                  # documento embutido
        "cidade": "SP",
        "uf": "SP"
    },
    "tags": ["python", "dados"],  # array
    "criado_em": datetime.now()
})

colecao.insert_many([
    {"nome": "Maria", "cidade": "RJ"},
    {"nome": "Carlos"}  # campos diferentes — tudo bem!
])
```

---

## Queries
```python
# Buscar um
usuario = colecao.find_one({"email": "joao@email.com"})

# Buscar vários
todos = list(colecao.find({"ativo": True}))

# Operadores de comparação
colecao.find({"idade": {"$gt": 18}})           # >
colecao.find({"idade": {"$gte": 18}})          # >=
colecao.find({"idade": {"$lt": 65}})           # <
colecao.find({"idade": {"$ne": 0}})            # !=
colecao.find({"idade": {"$in": [18, 25, 30]}}) # IN
colecao.find({"idade": {"$nin": [0, -1]}})     # NOT IN

# Operadores lógicos
colecao.find({"$and": [{"ativo": True}, {"cidade": "SP"}]})
colecao.find({"$or": [{"cidade": "SP"}, {"cidade": "RJ"}]})
colecao.find({"ativo": {"$not": {"$eq": False}}})

# Documentos embutidos
colecao.find({"endereco.cidade": "SP"})

# Arrays
colecao.find({"tags": "python"})              # contém "python"
colecao.find({"tags": {"$in": ["python", "sql"]}})
colecao.find({"tags": {"$all": ["python", "dados"]}})  # contém todos

# Regex
colecao.find({"nome": {"$regex": "^Jo", "$options": "i"}})

# Projeção (campos a retornar)
colecao.find({}, {"nome": 1, "email": 1, "_id": 0})

# Ordenar, limitar, pular
colecao.find().sort("nome", ASCENDING).limit(10).skip(20)
```

---

## Updates
```python
# Atualizar um
colecao.update_one(
    {"_id": ObjectId("...")},
    {"$set": {"ativo": True, "atualizado_em": datetime.now()}}
)

# Atualizar vários
colecao.update_many({"ativo": False}, {"$set": {"deletado": True}})

# Operadores de update
colecao.update_one({"nome": "João"}, {"$inc": {"pontos": 10}})        # incrementar
colecao.update_one({"nome": "João"}, {"$push": {"tags": "sql"}})      # adicionar ao array
colecao.update_one({"nome": "João"}, {"$pull": {"tags": "old_tag"}})  # remover do array
colecao.update_one({"nome": "João"}, {"$unset": {"campo": ""}})       # remover campo

# Upsert — insere se não existir
colecao.update_one(
    {"email": "novo@email.com"},
    {"$setOnInsert": {"criado_em": datetime.now()}, "$set": {"nome": "Novo"}},
    upsert=True
)
```

---

## Aggregation Pipeline
```python
pipeline = [
    # Filtrar
    {"$match": {"ativo": True, "criado_em": {"$gte": datetime(2024, 1, 1)}}},
    
    # Adicionar campos calculados
    {"$addFields": {"nome_upper": {"$toUpper": "$nome"}}},
    
    # Agrupar
    {"$group": {
        "_id": "$cidade",
        "total_usuarios": {"$sum": 1},
        "media_idade": {"$avg": "$idade"},
        "emails": {"$push": "$email"}    # lista de emails
    }},
    
    # Ordenar
    {"$sort": {"total_usuarios": -1}},
    
    # Limitar
    {"$limit": 10},
    
    # Lookup — equivalente ao JOIN
    {"$lookup": {
        "from": "pedidos",
        "localField": "_id",           # campo nesta coleção
        "foreignField": "cliente_id",  # campo na outra coleção
        "as": "pedidos"                # nome do campo resultante
    }},
    
    # Projeção final
    {"$project": {
        "cidade": "$_id",
        "total_usuarios": 1,
        "media_idade": {"$round": ["$media_idade", 2]},
        "_id": 0
    }}
]

resultado = list(colecao.aggregate(pipeline))
```

---

## Índices
```python
# Índice simples
colecao.create_index([("email", ASCENDING)], unique=True)

# Índice composto
colecao.create_index([("cidade", ASCENDING), ("idade", DESCENDING)])

# Índice de texto (full-text search)
colecao.create_index([("nome", "text"), ("bio", "text")])
colecao.find({"$text": {"$search": "engenheiro dados"}})

# TTL — documentos expiram automaticamente
colecao.create_index("criado_em", expireAfterSeconds=2592000)  # 30 dias

# Listar índices
colecao.list_indexes()
```

---

## Quando Usar MongoDB vs SQL
| Use MongoDB quando... | Use SQL quando... |
|----------------------|------------------|
| Schema muda com frequência | Schema é estável |
| Dados hierárquicos/documentos | Dados tabulares relacionais |
| Escalabilidade horizontal | Transações complexas entre tabelas |
| Prototipagem rápida | Integridade referencial crítica |
| Logs, eventos, IoT | Relatórios analíticos complexos |
