# 🐬 MySQL

> Banco de dados relacional open source mais popular do mundo. Muito usado em aplicações web.

**Relacionado:** [[Modelagem_de_Dados]] | [[PostgreSQL]] | [[SQLServer]] | [[SQLAlchemy]] | [[Docker]] | [[AWS]]

---

## Instalação
```bash
# Ubuntu
sudo apt install mysql-server
sudo systemctl start mysql
sudo mysql_secure_installation

# Docker
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=senha123 \
  -e MYSQL_DATABASE=meudb \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=senha123 \
  -p 3306:3306 \
  mysql:8.0
```

---

## Diferenças Principais do PostgreSQL
```sql
-- Auto incremento
id INT AUTO_INCREMENT PRIMARY KEY

-- Limite de resultados (mesmo que PostgreSQL)
SELECT * FROM tabela LIMIT 10 OFFSET 20;

-- Mostrar estrutura
SHOW DATABASES;
SHOW TABLES;
DESCRIBE usuarios;
SHOW CREATE TABLE usuarios;

-- Tipos de dados específicos
TINYINT         -- 0 a 255 ou -128 a 127
MEDIUMINT
ENUM('a','b')   -- tipo enumerado
SET('a','b')    -- conjunto de valores

-- String com charset
nome VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci

-- JSON (MySQL 5.7+)
dados JSON
SELECT JSON_EXTRACT(dados, '$.nome') FROM tabela;
SELECT dados->>'$.nome' FROM tabela;  -- MySQL 8.0+
```

---

## Engines de Armazenamento
| Engine | Uso |
|--------|-----|
| **InnoDB** (padrão) | Transações ACID, chaves estrangeiras |
| **MyISAM** | Apenas leitura, sem transações (legado) |
| **MEMORY** | Dados na RAM, temporários |

```sql
CREATE TABLE tabela (...) ENGINE=InnoDB;
```

---

## Conexão Python
```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://admin:senha@localhost:3306/meudb")

# Ou com PyMySQL direto
import pymysql
conn = pymysql.connect(host="localhost", user="admin", 
                       password="senha", database="meudb")
```
