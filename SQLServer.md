# 🏢 SQL Server

> Banco de dados relacional da Microsoft. Muito usado em empresas com infraestrutura Microsoft.

**Relacionado:** [[Modelagem_de_Dados]] | [[PostgreSQL]] | [[MySQL]] | [[SQLAlchemy]] | [[Windows]] | [[AWS]]

---

## Instalação
```bash
# Docker (mais prático para dev)
docker run -d \
  --name sqlserver \
  -e ACCEPT_EULA=Y \
  -e SA_PASSWORD=Senha@Forte123 \
  -p 1433:1433 \
  mcr.microsoft.com/mssql/server:2022-latest

# Conectar
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U SA -P 'Senha@Forte123'
```

---

## Diferenças Principais
```sql
-- TOP em vez de LIMIT
SELECT TOP 10 * FROM usuarios ORDER BY criado_em DESC;
SELECT TOP 10 PERCENT * FROM usuarios;  -- percentual

-- Identidade (auto incremento)
id INT IDENTITY(1,1) PRIMARY KEY

-- Paginação moderna
SELECT * FROM tabela
ORDER BY id
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Variáveis
DECLARE @data_inicio DATE = '2024-01-01';
DECLARE @total INT;
SELECT @total = COUNT(*) FROM pedidos WHERE criado_em >= @data_inicio;

-- IF/ELSE
IF @total > 1000
    PRINT 'Muitos pedidos'
ELSE
    PRINT 'Poucos pedidos'
```

---

## Schemas e Organização
```sql
-- Schemas para organizar objetos
CREATE SCHEMA vendas;
CREATE SCHEMA financeiro;

CREATE TABLE vendas.pedidos (...);
CREATE TABLE financeiro.lancamentos (...);

-- Referenciar com schema
SELECT * FROM vendas.pedidos;
```

---

## T-SQL — Transact-SQL
```sql
-- Stored Procedure
CREATE OR ALTER PROCEDURE sp_relatorio_vendas
    @data_inicio DATE,
    @data_fim DATE,
    @estado NVARCHAR(2) = NULL  -- parâmetro opcional
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT
        c.nome,
        SUM(p.valor) as total
    FROM pedidos p
    JOIN clientes c ON c.id = p.cliente_id
    WHERE p.criado_em BETWEEN @data_inicio AND @data_fim
      AND (@estado IS NULL OR c.estado = @estado)
    GROUP BY c.nome
    ORDER BY total DESC;
END;

-- Executar
EXEC sp_relatorio_vendas '2024-01-01', '2024-12-31', 'SP';

-- Função
CREATE FUNCTION fn_calcular_desconto(@valor DECIMAL(15,2), @pct DECIMAL(5,2))
RETURNS DECIMAL(15,2) AS
BEGIN
    RETURN @valor * (1 - @pct / 100);
END;

SELECT dbo.fn_calcular_desconto(1000, 10);  -- 900.00
```

---

## Conexão Python
```python
from sqlalchemy import create_engine

# Via pyodbc (driver ODBC)
engine = create_engine(
    "mssql+pyodbc://SA:Senha@Forte123@localhost:1433/meudb"
    "?driver=ODBC+Driver+17+for+SQL+Server"
)

# Instalar driver no Linux
# curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
# apt install msodbcsql17 unixodbc-dev
# pip install pyodbc
```

---

## SQL Server vs PostgreSQL

| Aspecto | SQL Server | PostgreSQL |
|---------|-----------|------------|
| Licença | Pago (Express gratuito) | Open source |
| OS | Windows/Linux | Multiplataforma |
| JSON | Básico | JSONB avançado |
| Window Functions | ✅ Completo | ✅ Completo |
| Integração .NET | Excelente | Boa |
| Custo em cloud | Alto | Baixo |
