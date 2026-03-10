# 📦 venv

> Módulo nativo do [[Python]] para criar ambientes virtuais. Simples e sem dependências externas.

**Relacionado:** [[Ambientes_Virtuais]] | [[pyenv]] | [[Poetry]] | [[Linux]] | [[Windows]]

---

## Criar e Ativar
```bash
# Criar (cria a pasta .venv no diretório atual)
python -m venv .venv

# Ativar
source .venv/bin/activate        # Linux / macOS
.venv\Scripts\activate           # Windows CMD
.venv\Scripts\Activate.ps1       # Windows PowerShell

# Verificar ativação
which python                     # Linux: deve mostrar .venv/bin/python
(nome do venv aparece no prompt) # (.venv) usuario@maquina:~/projeto$

# Desativar
deactivate
```

---

## Gerenciar Pacotes
```bash
# Instalar pacotes
pip install pandas
pip install "fastapi==0.111.0"     # versão específica
pip install "pandas>=2.0,<3.0"    # range de versão

# Instalar de arquivo
pip install -r requirements.txt

# Listar instalados
pip list
pip show pandas                    # detalhes de um pacote

# Salvar dependências
pip freeze > requirements.txt

# Atualizar
pip install --upgrade pandas
pip install --upgrade pip          # atualizar o próprio pip

# Remover
pip uninstall pandas
```

---

## requirements.txt
```text
# requirements.txt
pandas==2.2.0
sqlalchemy==2.0.25
requests==2.31.0
python-dotenv==1.0.0

# Com comentários e faixas de versão
# Processamento de dados
pandas>=2.0.0
numpy>=1.24.0

# Banco de dados
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0

# Ambiente
python-dotenv>=1.0.0
```

---

## Diferença entre pip freeze e pip list
```bash
pip list          # versão legível com formatação
pip freeze        # formato requirements.txt (para salvar)

# Exportar sem dependências de dev
pip freeze --exclude-editable > requirements.txt
```

---

## Recriar Ambiente
```bash
# Deletar e recriar
rm -rf .venv
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## venv vs outras ferramentas
| | venv | Poetry | conda |
|--|--|--|--|
| Nativo Python | ✅ | ❌ | ❌ |
| Gerencia versão Python | ❌ | ✅ | ✅ |
| Lock file | ❌ | ✅ | ✅ |
| Publicar pacotes | ❌ | ✅ | ❌ |
| Curva de aprendizado | Baixa | Média | Média |
