# 🐘 PostgreSQL

> Banco de dados relacional open source mais avançado. Principal escolha para engenharia de dados.

**Relacionado:** [[Modelagem_de_Dados]] | [[SQLAlchemy]] | [[Docker]] | [[Airflow]] | [[AWS]] | [[MySQL]]

---

## Instalação
```bash
# Ubuntu
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Via Docker (recomendado para dev)
docker run -d \
  --name postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=senha123 \
  -e POSTGRES_DB=meudb \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:16

# Conectar
sudo -u postgres psql          # Linux
psql -h localhost -U admin -d meudb  # com host
```

---

## Tipos de Dados
```sql
-- Numéricos
SMALLINT          -- -32768 a 32767
INTEGER / INT     -- -2B a 2B
BIGINT            -- -9.2 × 10^18 a 9.2 × 10^18
SERIAL            -- INTEGER auto-incremento
BIGSERIAL         -- BIGINT auto-incremento
NUMERIC(15, 2)    -- decimal exato (ideal para dinheiro)
REAL              -- float 4 bytes
DOUBLE PRECISION  -- float 8 bytes

-- Texto
VARCHAR(n)        -- tamanho variável com limite
TEXT              -- tamanho ilimitado
CHAR(n)           -- tamanho fixo

-- Data/Hora
DATE              -- data (sem hora)
TIME              -- hora (sem data)
TIMESTAMP         -- data + hora
TIMESTAMPTZ       -- timestamp com timezone (recomendado!)
INTERVAL          -- intervalo de tempo

-- Outros
BOOLEAN           -- TRUE / FALSE
UUID              -- identificador único universal
JSONB             -- JSON binário e indexável
JSON              -- JSON texto
ARRAY             -- arrays nativas: TEXT[], INT[]
```

---

## DDL — Criar Estruturas
```sql
CREATE TABLE pedidos (
    id          BIGSERIAL PRIMARY KEY,
    uuid        UUID DEFAULT gen_random_uuid() UNIQUE,
    cliente_id  INT NOT NULL REFERENCES clientes(id),
    status      VARCHAR(20) DEFAULT 'pendente'
                CHECK (status IN ('pendente','aprovado','cancelado')),
    valor       NUMERIC(15,2) NOT NULL CHECK (valor > 0),
    observacao  TEXT,
    criado_em   TIMESTAMPTZ DEFAULT NOW(),
    atualizado  TIMESTAMPTZ DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);
CREATE INDEX idx_pedidos_status ON pedidos(status) WHERE status != 'cancelado';

-- Alterar tabela
ALTER TABLE pedidos ADD COLUMN desconto NUMERIC(5,2) DEFAULT 0;
ALTER TABLE pedidos ALTER COLUMN status TYPE TEXT;
ALTER TABLE pedidos DROP COLUMN observacao;
ALTER TABLE pedidos RENAME COLUMN valor TO valor_bruto;
```

---

## DML — Manipular Dados
```sql
-- INSERT
INSERT INTO pedidos (cliente_id, valor) VALUES (1, 150.00);
INSERT INTO pedidos (cliente_id, valor) VALUES
    (1, 100.00),
    (2, 200.00),
    (3, 300.00);

-- INSERT ... ON CONFLICT (upsert)
INSERT INTO clientes (email, nome) VALUES ('joao@email.com', 'João')
ON CONFLICT (email) DO UPDATE SET nome = EXCLUDED.nome;

INSERT INTO clientes (email, nome) VALUES ('joao@email.com', 'João')
ON CONFLICT (email) DO NOTHING;

-- SELECT
SELECT * FROM pedidos WHERE status = 'pendente' ORDER BY criado_em DESC LIMIT 10;

-- UPDATE
UPDATE pedidos SET status = 'aprovado', atualizado = NOW() WHERE id = 1;

-- DELETE
DELETE FROM pedidos WHERE status = 'cancelado' AND criado_em < NOW() - INTERVAL '1 year';

-- RETURNING — retorna as linhas afetadas
INSERT INTO pedidos (cliente_id, valor) VALUES (1, 100) RETURNING id, uuid;
UPDATE pedidos SET status = 'aprovado' WHERE id = 1 RETURNING *;
```

---

## JOINs
```sql
-- INNER JOIN — apenas registros que casam
SELECT p.id, c.nome, p.valor
FROM pedidos p
INNER JOIN clientes c ON c.id = p.cliente_id;

-- LEFT JOIN — todos os da esquerda + matches da direita
SELECT c.nome, COUNT(p.id) as total_pedidos
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id
GROUP BY c.nome;

-- FULL OUTER JOIN — todos os registros de ambos os lados
SELECT * FROM tabela_a FULL OUTER JOIN tabela_b ON tabela_a.id = tabela_b.id;

-- CROSS JOIN — produto cartesiano (todas as combinações)
SELECT a.produto, b.cor FROM produtos a CROSS JOIN cores b;
```

---

## Funções Analíticas (Window Functions)
```sql
SELECT
    nome,
    departamento,
    salario,
    -- Agregação com OVER
    AVG(salario) OVER (PARTITION BY departamento) AS media_depto,
    SUM(salario) OVER (PARTITION BY departamento) AS total_depto,
    
    -- Ranking
    RANK() OVER (PARTITION BY departamento ORDER BY salario DESC) AS ranking,
    DENSE_RANK() OVER (ORDER BY salario DESC) AS ranking_geral,
    ROW_NUMBER() OVER (PARTITION BY departamento ORDER BY nome) AS linha,
    NTILE(4) OVER (ORDER BY salario) AS quartil,
    
    -- Deslocamento
    LAG(salario, 1) OVER (PARTITION BY departamento ORDER BY criado_em) AS salario_anterior,
    LEAD(salario, 1) OVER (PARTITION BY departamento ORDER BY criado_em) AS proximo_salario,
    FIRST_VALUE(salario) OVER (PARTITION BY departamento ORDER BY salario DESC) AS maior_salario,
    
    -- Acumulado
    SUM(salario) OVER (ORDER BY criado_em ROWS UNBOUNDED PRECEDING) AS acumulado
FROM funcionarios;
```

---

## CTEs
```sql
WITH 
-- CTE 1
clientes_ativos AS (
    SELECT id, nome FROM clientes WHERE ativo = TRUE
),
-- CTE 2 — referencia CTE 1
pedidos_recentes AS (
    SELECT p.cliente_id, SUM(p.valor) AS total_90d
    FROM pedidos p
    WHERE p.criado_em >= NOW() - INTERVAL '90 days'
    GROUP BY p.cliente_id
),
-- CTE recursiva (ex: hierarquias)
hierarquia AS (
    SELECT id, nome, gerente_id, 1 AS nivel
    FROM funcionarios WHERE gerente_id IS NULL
    UNION ALL
    SELECT f.id, f.nome, f.gerente_id, h.nivel + 1
    FROM funcionarios f
    JOIN hierarquia h ON h.id = f.gerente_id
)
SELECT ca.nome, COALESCE(pr.total_90d, 0) AS total
FROM clientes_ativos ca
LEFT JOIN pedidos_recentes pr ON pr.cliente_id = ca.id;
```

---

## Performance e EXPLAIN
```sql
-- Analisar plano de execução
EXPLAIN SELECT * FROM pedidos WHERE cliente_id = 1;
EXPLAIN ANALYZE SELECT * FROM pedidos WHERE cliente_id = 1;

-- Procurar por:
-- Seq Scan = leitura sequencial (lento para tabelas grandes — considere índice)
-- Index Scan = usando índice (rápido)
-- Rows = estimativa de linhas
-- cost = custo estimado (menor = melhor)
```
