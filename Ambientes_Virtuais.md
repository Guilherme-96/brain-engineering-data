# 🏗️ Ambientes Virtuais

> Isolamento de dependências por projeto. Essencial para evitar conflitos entre bibliotecas.

**Relacionado:** [[pyenv]] | [[venv]] | [[Poetry]] | [[VSCode]] | [[Linux]] | [[Windows]] | [[Docker]]

---

## Por que usar?
- Cada projeto pode ter versões **diferentes** da mesma biblioteca
- Evita o `pip install` global que quebra outros projetos
- Facilita a **reprodução** do ambiente em outra máquina
- Essencial para produção e times

---

## Ferramentas Disponíveis

| Ferramenta | Propósito | Quando usar |
|---|---|---|
| [[pyenv]] | Gerenciar versões do Python | Sempre — base de tudo |
| [[venv]] | Criar ambiente virtual | Projetos simples |
| [[Poetry]] | Gerenciar deps + venv + publicação | Projetos sérios/times |
| `conda` | Env + dados científicos | Data Science / ML |
| `pipenv` | venv + Pipfile | Alternativa ao Poetry |

---

## Fluxo Recomendado (pyenv + venv)
```bash
# 1. Instalar a versão Python desejada
pyenv install 3.12.7

# 2. Entrar na pasta do projeto
cd ~/Desktop/meu-projeto

# 3. Definir Python local para o projeto
pyenv local 3.12.7

# 4. Criar ambiente virtual
python -m venv .venv

# 5. Ativar
source .venv/bin/activate          # Linux/macOS
.venv\Scripts\activate             # Windows

# 6. Verificar
which python                       # deve apontar para .venv
python --version

# 7. Instalar dependências
pip install pandas sqlalchemy requests

# 8. Salvar dependências
pip freeze > requirements.txt

# 9. Desativar quando terminar
deactivate
```

---

## Reproduzir ambiente em outra máquina
```bash
# 1. Clonar o projeto
git clone https://github.com/usuario/projeto.git
cd projeto

# 2. Instalar a versão correta do Python (se tiver .python-version)
pyenv install $(cat .python-version)

# 3. Criar ambiente virtual
python -m venv .venv
source .venv/bin/activate

# 4. Instalar exatamente as mesmas versões
pip install -r requirements.txt
```

---

## O que NÃO commitar
```gitignore
# .gitignore
.venv/
venv/
env/
__pycache__/
*.pyc
.python-version    # opcional — commitar ajuda a padronizar o time
```

---

## Dica Visual no VS Code
Com a extensão **Python** instalada:
- `Ctrl+Shift+P` → "Python: Select Interpreter"
- Escolha o Python dentro de `.venv`
- O terminal integrado já vai ativar automaticamente
