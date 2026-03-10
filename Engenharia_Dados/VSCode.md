# 💻 VS Code

> Editor de código leve, poderoso e extensível. O mais popular entre engenheiros de dados.

**Relacionado:** [[Python]] | [[Ambientes_Virtuais]] | [[Linux]] | [[Windows]] | [[Git_GitHub]] | [[Docker]]

---

## Instalação
```bash
# Linux (Ubuntu/Debian)
sudo snap install code --classic

# Ou via apt
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -D -o root -g root -m 644 microsoft.gpg /etc/apt/keyrings/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,signed-by=/etc/apt/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update && sudo apt install code

# Windows: https://code.visualstudio.com/download
# macOS: brew install --cask visual-studio-code
```

---

## Extensões Essenciais para Engenharia de Dados
```
# Python
ms-python.python              Python — suporte completo
ms-python.pylance             Pylance — IntelliSense avançado
ms-python.black-formatter     Black — formatação automática
ms-python.mypy-type-checker   MyPy — verificação de tipos

# Dados
mechatroner.rainbow-csv       Rainbow CSV — visualizar CSV colorido
GrapeCity.gc-excelviewer      Excel Viewer — visualizar xlsx

# Git
eamodio.gitlens               GitLens — histórico inline, blame

# Containers
ms-azuretools.vscode-docker   Docker — gerenciar containers

# Remote
ms-vscode-remote.remote-wsl   Remote WSL — dev no WSL (Windows)
ms-vscode-remote.remote-ssh   Remote SSH — dev em servidor remoto

# Banco de dados
cweijan.vscode-database-client Database Client — SQL no editor

# Qualidade de código
streetsidesoftware.code-spell-checker  Spell Checker
oderwat.indent-rainbow         Indent Rainbow — visualizar identação
```

---

## Atalhos Essenciais

### Navegação
| Atalho | Ação |
|--------|------|
| `Ctrl+P` | Abrir arquivo por nome |
| `Ctrl+Shift+P` | Paleta de comandos |
| `Ctrl+G` | Ir para linha |
| `Ctrl+Tab` | Alternar entre arquivos abertos |
| `Ctrl+\` | Dividir editor |
| `Ctrl+B` | Toggle sidebar |
| `` Ctrl+` `` | Abrir/fechar terminal |

### Edição
| Atalho | Ação |
|--------|------|
| `Ctrl+D` | Selecionar próxima ocorrência |
| `Ctrl+Shift+L` | Selecionar todas as ocorrências |
| `Alt+↑/↓` | Mover linha |
| `Shift+Alt+↑/↓` | Duplicar linha |
| `Ctrl+Shift+K` | Deletar linha |
| `Ctrl+/` | Comentar/descomentar |
| `Ctrl+Shift+F` | Buscar em todos os arquivos |
| `F2` | Renomear símbolo (variável, função) |
| `F12` | Ir para definição |
| `Alt+F12` | Peek definition (ver sem sair) |
| `Shift+F12` | Ver todas as referências |

### Python específico
| Atalho | Ação |
|--------|------|
| `Shift+Enter` | Rodar seleção no terminal |
| `F5` | Debug |
| `Ctrl+Shift+P` → "Select Interpreter" | Escolher Python/venv |

---

## settings.json Recomendado
```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "editor.formatOnSave": true,
  "editor.rulers": [88],
  "editor.tabSize": 4,
  "editor.insertSpaces": true,
  "editor.wordWrap": "on",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "files.exclude": {
    "**/__pycache__": true,
    "**/*.pyc": true,
    "**/.pytest_cache": true
  },
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": true,
  "terminal.integrated.defaultProfile.linux": "bash",
  "git.autofetch": true
}
```

---

## Dicas de Produtividade
- **Multi-cursor:** `Alt+Click` para adicionar cursores em múltiplas linhas
- **Renomear variável:** `F2` renomeia em todos os arquivos do projeto
- **Terminal integrado:** `` Ctrl+` `` abre terminal já na pasta do projeto
- **Zen Mode:** `Ctrl+K Z` para focar apenas no código
- **Live Share:** colaboração em tempo real (extensão Microsoft)
