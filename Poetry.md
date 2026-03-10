# 📜 Poetry

> Gerenciador moderno de dependências e ambientes virtuais para [[Python]]. Ideal para times e projetos sérios.

**Relacionado:** [[Ambientes_Virtuais]] | [[venv]] | [[pyenv]] | [[Git_GitHub]]

**Instalação:**
```bash
curl -sSL https://install.python-poetry.org | python3 -
# Adicionar ao PATH conforme instrução da instalação
```

---

## Criar Projeto
```bash
# Novo projeto (cria estrutura completa)
poetry new meu-projeto
# meu-projeto/
# ├── pyproject.toml
# ├── README.md
# ├── meu_projeto/
# │   └── __init__.py
# └── tests/
#     └── __init__.py

# Iniciar em projeto existente
cd projeto-existente
poetry init        # cria pyproject.toml interativamente
```

---

## Gerenciar Dependências
```bash
# Adicionar
poetry add pandas
poetry add "pandas>=2.0"
poetry add requests sqlalchemy       # múltiplos

# Dependência de desenvolvimento
poetry add pytest black mypy --group dev

# Remover
poetry remove pandas

# Atualizar
poetry update                        # tudo
poetry update pandas                 # específico

# Instalar (do pyproject.toml + poetry.lock)
poetry install
poetry install --without dev         # sem deps de dev (produção)
```

---

## pyproject.toml
```toml
[tool.poetry]
name = "meu-pipeline"
version = "0.1.0"
description = "Pipeline de dados"
authors = ["Seu Nome <seu@email.com>"]

[tool.poetry.dependencies]
python = "^3.12"
pandas = "^2.0"
sqlalchemy = "^2.0"
requests = "^2.31"
python-dotenv = "^1.0"

[tool.poetry.group.dev.dependencies]
pytest = "^8.0"
black = "^24.0"
mypy = "^1.8"
pytest-cov = "^4.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

## Usar o Ambiente
```bash
# Ativar o shell com o ambiente
poetry shell

# Rodar sem ativar
poetry run python script.py
poetry run pytest
poetry run black .

# Informações do ambiente
poetry env info
poetry env list
```

---

## poetry.lock
- Arquivo gerado automaticamente — **sempre commitar!**
- Garante que todos do time usem **exatamente** as mesmas versões
- `poetry install` usa o lock file para reprodutibilidade

---

## Comparação com pip + venv
```bash
# Com pip + venv
python -m venv .venv
source .venv/bin/activate
pip install pandas requests
pip freeze > requirements.txt
# requirements.txt pode ter versões inconsistentes entre máquinas

# Com Poetry
poetry add pandas requests
# poetry.lock garante reprodutibilidade 100%
```
