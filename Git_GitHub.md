# 🔀 Git e GitHub

> Controle de versão distribuído. Essencial para qualquer projeto de software ou dados.

**Relacionado:** [[Linux]] | [[VSCode]] | [[Python]] | [[Docker]] | [[Terraform]]

---

## Configuração Inicial
```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
git config --global core.editor "code --wait"   # VS Code como editor
git config --global init.defaultBranch main
git config --list
```

---

## Fluxo Básico Diário
```bash
# Iniciar
git init
git clone https://github.com/usuario/repo.git

# Ver estado
git status
git diff                  # mudanças não staged
git diff --staged         # mudanças staged (prontas para commit)

# Adicionar ao stage
git add arquivo.py        # arquivo específico
git add .                 # tudo
git add -p                # interativo (escolhe por pedaços)

# Commit
git commit -m "feat: adiciona pipeline de ETL"
git commit --amend        # editar mensagem do último commit

# Enviar / Receber
git push origin main
git push -u origin main   # define upstream (primeira vez)
git pull origin main
git fetch origin          # baixa sem fazer merge
```

---

## Branches
```bash
git branch                     # listar branches locais
git branch -a                  # locais + remotas
git branch feature/nova        # criar branch
git checkout feature/nova      # trocar de branch
git checkout -b feature/nova   # criar e trocar de uma vez
git switch feature/nova        # comando mais novo (recomendado)
git switch -c feature/nova     # criar e trocar

# Merge
git checkout main
git merge feature/nova         # merge na branch atual
git merge --no-ff feature/nova # sempre cria commit de merge

# Rebase (histórico mais limpo)
git checkout feature/nova
git rebase main                # rebasa feature sobre main

# Deletar branch
git branch -d feature/nova     # seguro (só se merged)
git branch -D feature/nova     # forçado
git push origin --delete feature/nova  # deletar remota
```

---

## Histórico
```bash
git log                           # histórico completo
git log --oneline                 # resumido
git log --oneline --graph         # com grafo de branches
git log --author="Nome"
git log --since="2024-01-01"
git log -- arquivo.py             # histórico de um arquivo
git show abc1234                  # detalhes de um commit
git blame arquivo.py              # quem editou cada linha
```

---

## Desfazer Coisas
```bash
# Descartar mudanças no arquivo (não staged)
git restore arquivo.py

# Remover do stage (manter mudanças)
git restore --staged arquivo.py

# Desfazer último commit (mantém mudanças staged)
git reset --soft HEAD~1

# Desfazer último commit (mantém mudanças não staged)
git reset HEAD~1

# Desfazer e jogar fora as mudanças — CUIDADO!
git reset --hard HEAD~1

# Criar novo commit que desfaz um anterior (seguro)
git revert abc1234

# Stash — guardar mudanças temporariamente
git stash                     # guarda
git stash list                # listar
git stash pop                 # aplica e remove
git stash apply stash@{0}     # aplica sem remover
git stash drop stash@{0}      # descarta
```

---

## Tags
```bash
git tag v1.0.0                              # tag leve
git tag -a v1.0.0 -m "Release 1.0.0"      # tag anotada
git push origin v1.0.0                     # enviar tag
git push origin --tags                     # enviar todas as tags
git tag -d v1.0.0                          # deletar local
```

---

## .gitignore para Projetos Python
```gitignore
# Ambientes virtuais
.venv/
venv/
env/

# Python
__pycache__/
*.pyc
*.pyo
*.egg-info/
dist/
build/
.eggs/

# IDEs
.vscode/settings.json
.idea/
*.swp
*.swo

# Dados sensíveis
.env
*.env
secrets.yml
credentials.json
*.pem
*.key

# Dados (geralmente grandes demais para git)
data/raw/
data/processed/
*.csv
*.parquet
*.xlsx

# Logs
logs/
*.log

# Airflow
airflow.db
airflow.cfg
```

---

## Conventional Commits
```
Formato: tipo(escopo): descrição

Tipos:
feat:     nova funcionalidade
fix:      correção de bug
docs:     apenas documentação
style:    formatação, sem mudança de lógica
refactor: refatoração sem feat ou fix
test:     adição ou correção de testes
chore:    manutenção, dependências
ci:       mudanças em CI/CD
perf:     melhoria de performance

Exemplos:
feat(pipeline): adiciona ingestão de dados do MongoDB
fix(etl): corrige tratamento de valores nulos em datas
docs(readme): atualiza instruções de instalação
chore(deps): atualiza pandas para 2.2.0
```

---

## GitHub — Funcionalidades Chave
- **Issues:** rastrear bugs e tarefas
- **Pull Requests:** revisão de código antes de merge
- **Actions:** CI/CD — rodar testes, builds e deploys automaticamente
- **Projects:** kanban para organizar issues
- **Releases:** versionamento de entregas com changelog
- **Secrets:** armazenar tokens e senhas para Actions

```yaml
# .github/workflows/testes.yml — exemplo de CI
name: Testes

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest --cov=src
```
