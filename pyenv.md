# 🐍 pyenv

> Gerenciador de versões do Python. Permite ter múltiplas versões instaladas e trocar facilmente.

**Relacionado:** [[Ambientes_Virtuais]] | [[venv]] | [[Linux]] | [[Windows]]

---

## Instalação no Linux/macOS
```bash
# 1. Dependências (Ubuntu/Debian)
sudo apt update && sudo apt install -y \
  build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev \
  libncursesw5-dev xz-utils tk-dev \
  libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# 2. Instalar pyenv
curl https://pyenv.run | bash

# 3. Adicionar ao ~/.bashrc
echo '
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
' >> ~/.bashrc

source ~/.bashrc
```

---

## Comandos Essenciais
```bash
# Versões disponíveis para instalar
pyenv install --list
pyenv install --list | grep "3.12"

# Instalar versão
pyenv install 3.12.7
pyenv install 3.11.9

# Listar versões instaladas
pyenv versions
# * 3.14.0 (global)
#   3.12.7
#   3.11.9

# Definir versão GLOBAL (padrão do sistema)
pyenv global 3.12.7

# Definir versão LOCAL (só na pasta atual → cria .python-version)
cd ~/Desktop/meu-projeto
pyenv local 3.12.7

# Definir versão só na sessão atual do terminal
pyenv shell 3.11.9

# Ver qual Python está ativo
pyenv version
which python
python --version

# Remover versão
pyenv uninstall 3.11.9
```

---

## Hierarquia de Prioridade
```
shell > local > global

1. pyenv shell  → vale só no terminal atual
2. pyenv local  → vale na pasta (lê .python-version)
3. pyenv global → vale em todo o sistema
```

---

## Atualizar pyenv
```bash
pyenv update
```

---

## Problema comum: Permission denied
```bash
# ERRADO: rodar pyenv local na raiz /
cd /
pyenv local 3.12.7   # Permission denied!

# CORRETO: sempre dentro da pasta do projeto
cd ~/Desktop/meu-projeto
pyenv local 3.12.7   # ✅
```
