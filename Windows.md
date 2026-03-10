# 🪟 Windows

> Sistema operacional para desenvolvimento com foco em PowerShell, WSL2 e ferramentas de dados.

**Relacionado:** [[Linux]] | [[VSCode]] | [[Docker]] | [[Python]] | [[Ambientes_Virtuais]]

---

## PowerShell Essencial
```powershell
# Navegação
Get-Location              # pwd
Set-Location C:\projetos  # cd
Get-ChildItem             # ls / dir
Get-ChildItem -Recurse -Filter "*.csv"

# Criar
New-Item arquivo.txt -ItemType File
New-Item pasta -ItemType Directory
New-Item caminho\longo -ItemType Directory -Force  # cria toda estrutura

# Copiar / Mover
Copy-Item origem.txt destino.txt
Copy-Item pasta\ destino\ -Recurse
Move-Item arquivo.txt .\nova_pasta\
Rename-Item arquivo.txt novo_nome.txt

# Remover
Remove-Item arquivo.txt
Remove-Item pasta\ -Recurse -Force

# Conteúdo
Get-Content arquivo.txt           # cat
Get-Content arquivo.txt -Head 20  # head
Get-Content arquivo.txt -Tail 20  # tail

# Busca
Select-String -Path *.log -Pattern "ERROR"
Get-ChildItem -Recurse | Where-Object { $_.Name -like "*.py" }
```

---

## Variáveis de Ambiente
```powershell
# Ver variável
$env:PATH
$env:PYTHONPATH

# Definir para a sessão atual
$env:MINHA_VAR = "valor"

# Definir permanentemente
[System.Environment]::SetEnvironmentVariable("MINHA_VAR", "valor", "User")     # usuário
[System.Environment]::SetEnvironmentVariable("MINHA_VAR", "valor", "Machine")  # sistema (admin)

# Adicionar ao PATH permanentemente
$oldPath = [System.Environment]::GetEnvironmentVariable("PATH", "User")
[System.Environment]::SetEnvironmentVariable("PATH", "$oldPath;C:\novo\caminho", "User")
```

---

## Chocolatey — Gerenciador de Pacotes
```powershell
# Instalar Chocolatey (como Administrador)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# Instalar pacotes
choco install git
choco install vscode
choco install python
choco install docker-desktop
choco install nodejs
choco install 7zip
choco install googlechrome

# Atualizar
choco upgrade all
choco upgrade git
```

---

## WSL2 — Windows Subsystem for Linux
```powershell
# Instalar WSL2 (PowerShell como Admin)
wsl --install                           # instala Ubuntu por padrão
wsl --install -d Ubuntu-22.04           # versão específica
wsl --list --verbose                    # listar distribuições
wsl --set-default Ubuntu-22.04
wsl --set-version Ubuntu-22.04 2        # garantir WSL2 (não WSL1)

# Iniciar
wsl                                     # abre a distro padrão
wsl -d Ubuntu-22.04                     # distro específica

# Acessar arquivos Windows pelo WSL
ls /mnt/c/Users/SeuUsuario/Desktop/

# Acessar arquivos WSL pelo Windows Explorer
# \\wsl$\Ubuntu-22.04\home\usuario\
```

---

## Python no Windows
```powershell
# Instalar Python (Microsoft Store ou python.org)
winget install Python.Python.3.12

# Ou via Chocolatey
choco install python --version=3.12.7

# pyenv-win
Invoke-WebRequest -UseBasicParsing `
  -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" `
  -OutFile "./install-pyenv-win.ps1"
.\install-pyenv-win.ps1

pyenv install 3.12.7
pyenv global 3.12.7

# Criar venv no Windows
python -m venv .venv
.venv\Scripts\activate
pip install pandas
```

---

## Atalhos do Windows
| Atalho | Ação |
|--------|------|
| `Win+E` | Abrir Explorer |
| `Win+R` | Executar |
| `Win+X` | Menu de administração |
| `Ctrl+Shift+Esc` | Gerenciador de Tarefas |
| `Win+.` | Emoji picker |
| `Win+V` | Histórico da área de transferência |
| `Win+Shift+S` | Screenshot recortado |
| `F2` | Renomear arquivo |
| `Alt+F4` | Fechar janela |

---

## Terminal Windows (Windows Terminal)
```
Instalar: Microsoft Store → "Windows Terminal"

Configurar padrão:
- Configurações → Inicialização → Perfil padrão → Ubuntu (WSL)
- Fica fácil alternar entre PowerShell, CMD e Linux
```
