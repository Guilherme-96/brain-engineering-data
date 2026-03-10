# 📐 Modelagem de Dados

> Fundamentos para estruturar dados de forma eficiente. Base para [[PostgreSQL]], [[MySQL]], [[SQLServer]] e Data Warehouses.

**Relacionado:** [[PostgreSQL]] | [[MySQL]] | [[SQLServer]] | [[MongoDB]] | [[Pandas]] | [[Spark_PySpark]]

---

## Formas Normais (Normalização)

### 1FN — Primeira Forma Normal
- Atributos **atômicos** (indivisíveis)
- Sem grupos repetitivos

```
❌ Antes:
| id | nome  | telefones              |
|----|-------|------------------------|
| 1  | João  | (11)1111, (11)2222     |

✅ Depois (1FN):
| id | nome  | telefone   |
|----|-------|------------|
| 1  | João  | (11)1111   |
| 1  | João  | (11)2222   |
```

### 2FN — Segunda Forma Normal
- Estar em 1FN
- Sem dependências **parciais** de chave composta

```
❌ Chave: (pedido_id, produto_id) — nome_produto depende só de produto_id
| pedido_id | produto_id | nome_produto | quantidade |

✅ Separar em duas tabelas
| pedido_id | produto_id | quantidade |
| produto_id | nome_produto |
```

### 3FN — Terceira Forma Normal
- Estar em 2FN
- Sem dependências **transitivas** (atributo não-chave dependendo de outro não-chave)

```
❌ cidade depende de cep, que depende de id
| id | cep   | cidade     |

✅ Separar:
| id | cep   |
| cep | cidade |
```

---

## Modelagem Dimensional (Data Warehouse)

### Star Schema — Esquema Estrela
```
Tabela FATO (centro): métricas e chaves estrangeiras
Tabelas DIMENSÃO (pontas): atributos descritivos

         dim_tempo
             |
dim_produto──fato_vendas──dim_cliente
             |
          dim_loja
```

```sql
-- Tabela Fato
CREATE TABLE fato_vendas (
    id_venda      BIGSERIAL PRIMARY KEY,
    id_tempo      INT REFERENCES dim_tempo(id),
    id_produto    INT REFERENCES dim_produto(id),
    id_cliente    INT REFERENCES dim_cliente(id),
    id_loja       INT REFERENCES dim_loja(id),
    quantidade    INT,
    valor_total   NUMERIC(15,2),
    custo_total   NUMERIC(15,2)
);

-- Tabela Dimensão
CREATE TABLE dim_produto (
    id            SERIAL PRIMARY KEY,
    nome          VARCHAR(200),
    categoria     VARCHAR(100),
    subcategoria  VARCHAR(100),
    marca         VARCHAR(100),
    preco_lista   NUMERIC(15,2)
);

CREATE TABLE dim_tempo (
    id            INT PRIMARY KEY,   -- YYYYMMDD como inteiro
    data          DATE,
    ano           INT,
    semestre      INT,
    trimestre     INT,
    mes           INT,
    nome_mes      VARCHAR(20),
    semana        INT,
    dia           INT,
    dia_semana    INT,
    nome_dia      VARCHAR(20),
    eh_feriado    BOOLEAN,
    eh_fim_semana BOOLEAN
);
```

### Snowflake Schema — Esquema Floco de Neve
- Dimensões **normalizadas** (subdimensões)
- Mais complexo, mas ocupa menos espaço

```
dim_produto → dim_categoria → dim_departamento
```

---

## SCD — Slowly Changing Dimensions

### Tipo 1 — Sobrescrever (sem histórico)
```sql
UPDATE dim_cliente SET email = 'novo@email.com' WHERE id = 1;
-- Perde o histórico do email antigo
```

### Tipo 2 — Nova linha (com histórico completo)
```sql
-- Adicionar nova linha com flags de vigência
ALTER TABLE dim_cliente ADD COLUMN data_inicio DATE;
ALTER TABLE dim_cliente ADD COLUMN data_fim DATE;
ALTER TABLE dim_cliente ADD COLUMN is_current BOOLEAN;

-- Quando cliente muda de cidade:
UPDATE dim_cliente SET data_fim = CURRENT_DATE, is_current = FALSE WHERE id = 1 AND is_current = TRUE;
INSERT INTO dim_cliente (nome, email, cidade, data_inicio, data_fim, is_current)
VALUES ('João', 'joao@email.com', 'RJ', CURRENT_DATE, NULL, TRUE);
```

### Tipo 3 — Coluna adicional (histórico limitado)
```sql
ALTER TABLE dim_cliente ADD COLUMN cidade_anterior VARCHAR(100);
UPDATE dim_cliente SET cidade_anterior = cidade, cidade = 'RJ' WHERE id = 1;
```

---

## Índices — Estratégia

### Quando criar
```sql
-- Colunas usadas em WHERE frequentemente
CREATE INDEX idx_pedidos_status ON pedidos(status);

-- Colunas usadas em JOIN
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);

-- Colunas usadas em ORDER BY
CREATE INDEX idx_pedidos_data ON pedidos(criado_em DESC);

-- Índice composto (ordem importa!)
CREATE INDEX idx_vendas_cliente_data ON vendas(cliente_id, criado_em);
-- Funciona para: WHERE cliente_id = ?
-- Funciona para: WHERE cliente_id = ? AND criado_em > ?
-- NÃO funciona para: WHERE criado_em > ? (sem cliente_id)
```

### Tipos de índice (PostgreSQL)
```sql
-- B-Tree (padrão) — ideal para =, <, >, BETWEEN, ORDER BY
CREATE INDEX idx_nome ON tabela(nome);

-- Hash — ideal apenas para igualdade (=)
CREATE INDEX idx_hash ON tabela USING HASH (email);

-- GIN — ideal para arrays, JSONB, full-text search
CREATE INDEX idx_gin ON produtos USING GIN(tags);
CREATE INDEX idx_fts ON artigos USING GIN(to_tsvector('portuguese', conteudo));

-- BRIN — ideal para tabelas muito grandes com dados sequenciais (logs, IoT)
CREATE INDEX idx_brin ON eventos USING BRIN(criado_em);
```

---

## Data Lakehouse — Arquitetura Moderna

```
Camadas:
┌─────────────────────────────────┐
│  Landing / Bronze                │  Dados brutos, como vieram
├─────────────────────────────────┤
│  Silver                          │  Limpos, validados, tipados
├─────────────────────────────────┤
│  Gold                            │  Agregados, prontos para negócio
└─────────────────────────────────┘

Formatos recomendados:
- Bronze: JSON, CSV, Parquet
- Silver: Parquet, Delta Lake
- Gold: Parquet, Delta Lake, tabelas analíticas
```
