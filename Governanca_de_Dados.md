# 🏛️ Governança de Dados

> Conjunto de práticas, processos e políticas para garantir que os dados sejam confiáveis, seguros e bem utilizados.

**Relacionado:** [[Modelagem_de_Dados]] | [[Observabilidade_Pipelines]] | [[AWS]] | [[Azure]] | [[Databricks]] | [[dbt]] | [[PostgreSQL]]

---

## O que é Governança de Dados?

> "Dados sem governança são como uma biblioteca sem catálogo — os livros existem, mas ninguém sabe onde estão ou se ainda são válidos."

Governança de dados engloba:
- **Quem** pode acessar quais dados
- **O quê** cada dado significa (glossário de negócio)
- **Como** os dados foram gerados e transformados (lineage)
- **Quando** os dados foram atualizados e com qual qualidade
- **Por que** certos dados existem (conformidade e regulação)

---

## Pilares da Governança

### 1. Catálogo de Dados (Data Catalog)
```
Um "Google interno" para os dados da empresa.

Ferramentas:
- Apache Atlas (open source)
- DataHub (LinkedIn, open source) ← muito usado
- AWS Glue Data Catalog
- Azure Purview
- Collibra (enterprise)
- Alation (enterprise)

O catálogo registra:
- Onde cada dado está (banco, tabela, coluna)
- O que significa cada campo (descrição de negócio)
- Quem é o dono (data owner)
- Quando foi atualizado
- De onde veio (lineage)
```

### 2. Qualidade de Dados (Data Quality)
```python
# Dimensões de qualidade:
# Completude    — percentual de campos não nulos
# Unicidade     — sem duplicatas
# Consistência  — valores coerentes entre sistemas
# Acurácia      — valor correto (difícil de medir)
# Timeliness    — dado atualizado no tempo esperado
# Validade      — segue as regras de negócio

# Ferramentas:
# Great Expectations — ver Observabilidade_Pipelines
# dbt tests          — ver dbt
# Soda Core          — open source

# Exemplo com Great Expectations
import great_expectations as gx

context = gx.get_context()
validator = context.sources.pandas_default.read_csv("clientes.csv")

validator.expect_column_values_to_not_be_null("email")
validator.expect_column_values_to_be_unique("cpf")
validator.expect_column_values_to_match_regex("cpf", r"^\d{11}$")
validator.expect_column_values_to_be_between("idade", min_value=0, max_value=120)

results = validator.validate()
print(results.success)  # True ou False
```

### 3. Linhagem de Dados (Data Lineage)
```
Rastrear o caminho do dado desde a origem até o consumo:

[Sistema CRM] → [Pipeline ETL] → [PostgreSQL raw] → [dbt transform] → [tabela gold] → [Power BI]

Se a tabela do Power BI estiver errada → você sabe exatamente onde investigar.

Ferramentas:
- dbt (lineage automático via grafo)
- OpenLineage + Marquez
- Apache Atlas
- DataHub
```

### 4. Controle de Acesso (RBAC)
```sql
-- PostgreSQL — controle granular de acesso
-- Criar roles
CREATE ROLE analista_leitura;
CREATE ROLE engenheiro_dados;
CREATE ROLE admin_dados;

-- Permissões por role
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analista_leitura;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO engenheiro_dados;
GRANT ALL PRIVILEGES ON DATABASE meudb TO admin_dados;

-- Row Level Security — controle por linha!
ALTER TABLE pedidos ENABLE ROW LEVEL SECURITY;

CREATE POLICY visualizar_propria_regiao ON pedidos
    FOR SELECT
    USING (regiao = current_setting('app.user_regiao'));

-- Usuário só vê pedidos da sua região
SET app.user_regiao = 'SP';
SELECT * FROM pedidos;  -- retorna só pedidos de SP
```

### 5. Privacidade e LGPD/GDPR
```python
# Mascaramento de dados sensíveis
import hashlib
import re

def mascarar_cpf(cpf: str) -> str:
    """Mantém formato mas mascara dígitos."""
    return re.sub(r'\d', '*', cpf[:-2]) + cpf[-2:]

def anonimizar_email(email: str) -> str:
    """Hash irreversível do email."""
    return hashlib.sha256(email.encode()).hexdigest()

def pseudonimizar_nome(nome: str, chave: str) -> str:
    """Pseudonimização — reversível com a chave."""
    import hmac
    return hmac.new(chave.encode(), nome.encode(), hashlib.sha256).hexdigest()[:12]

# Categorias de dados pessoais (LGPD)
DADOS_SENSIVEIS = ["cpf", "rg", "cnh", "email", "telefone",
                   "endereco", "data_nascimento", "biometria",
                   "origem_racial", "religiao", "saude"]

# Regra de retenção — apagar dados após prazo
# DELETE FROM usuarios WHERE deletado_em < NOW() - INTERVAL '5 years';
```

---

## DataOps — Governança em Pipelines

```
DevOps aplicado a dados:

Versionamento de código    → Git (ver Git_GitHub)
Versionamento de esquemas  → Flyway, Liquibase, dbt
Testes automatizados       → Great Expectations, dbt tests
CI/CD de pipelines         → GitHub Actions + dbt Cloud
Monitoramento              → ver Observabilidade_Pipelines
Documentação               → dbt docs
```

---

## Mesh de Dados (Data Mesh)

Arquitetura moderna para governança em escala:

```
Conceito: descentralizar a propriedade dos dados por domínio

Time de Vendas    → dono dos dados de vendas
Time de Marketing → dono dos dados de campanhas
Time de Produto   → dono dos dados de uso do produto

Princípios:
1. Dados como produto (cada domínio é responsável pela qualidade)
2. Propriedade descentralizada por domínio
3. Plataforma de dados como infraestrutura compartilhada
4. Governança federada (padrões globais, execução local)
```

---

## Ferramentas por Tamanho de Empresa

| Porte | Catálogo | Qualidade | Lineage |
|-------|---------|-----------|---------|
| Startup | dbt docs | dbt tests | dbt |
| Médio | DataHub | Great Expectations + dbt | DataHub |
| Enterprise | Collibra / Purview | Soda + GE | Apache Atlas |
