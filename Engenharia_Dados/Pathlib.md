# 📁 Pathlib

> Manipulação de caminhos de arquivos orientada a objetos. Parte da biblioteca padrão do [[Python]] (3.4+).

**Relacionado:** [[Python]] | [[Linux]] | [[Windows]] | [[Pandas]]

---

## Criando Caminhos
```python
from pathlib import Path

# Caminhos absolutos e relativos
base = Path("/home/usuario/projetos")
relativo = Path("dados/arquivo.csv")

# Concatenar com operador /
arquivo = base / "dados" / "arquivo.csv"
# /home/usuario/projetos/dados/arquivo.csv

# Expandir ~
home = Path("~/Desktop").expanduser()
# /home/usuario/Desktop

# Diretório atual
atual = Path.cwd()
```

---

## Inspecionar Caminhos
```python
p = Path("/home/usuario/dados/vendas.csv")

p.name       # "vendas.csv"
p.stem       # "vendas"        (sem extensão)
p.suffix     # ".csv"          (extensão)
p.suffixes   # [".csv"]        (múltiplas extensões)
p.parent     # /home/usuario/dados
p.parents[0] # /home/usuario/dados
p.parents[1] # /home/usuario
p.parts      # ('/', 'home', 'usuario', 'dados', 'vendas.csv')

p.exists()   # True/False
p.is_file()  # True/False
p.is_dir()   # True/False
p.stat().st_size      # tamanho em bytes
p.stat().st_mtime     # data de modificação
```

---

## Criar e Remover
```python
# Criar diretórios
Path("nova/pasta/aninhada").mkdir(parents=True, exist_ok=True)

# Criar arquivo vazio
Path("arquivo.txt").touch()

# Remover
Path("arquivo.txt").unlink()              # remove arquivo
Path("pasta_vazia").rmdir()               # remove pasta vazia
import shutil
shutil.rmtree("pasta_com_conteudo/")      # remove pasta com conteúdo
```

---

## Listagem e Busca
```python
base = Path("/home/usuario/projetos")

# Listar diretório (não recursivo)
for item in base.iterdir():
    print(item)

# Glob — busca por padrão
list(base.glob("*.csv"))          # CSVs no diretório atual
list(base.glob("**/*.csv"))       # CSVs em todos os subdiretórios
list(base.rglob("*.parquet"))     # rglob = glob recursivo

# Filtrar por tipo
csvs = [f for f in base.rglob("*") if f.is_file() and f.suffix == ".csv"]
```

---

## Leitura e Escrita
```python
p = Path("arquivo.txt")

# Texto
conteudo = p.read_text(encoding="utf-8")
p.write_text("novo conteúdo", encoding="utf-8")

# Bytes
dados = p.read_bytes()
p.write_bytes(b"dados binarios")
```

---

## Renomear e Mover
```python
p = Path("arquivo.csv")

# Renomear
novo = p.rename(p.parent / "novo_nome.csv")

# Mover para outra pasta
p.rename(Path("/outra/pasta") / p.name)

# Copiar (usando shutil)
import shutil
shutil.copy2(p, destino)        # copia preservando metadados
shutil.move(str(p), str(destino))
```

---

## Vantagem sobre os.path
```python
# Jeito antigo (os.path)
import os
caminho = os.path.join("/home", "usuario", "dados", "arquivo.csv")
nome = os.path.basename(caminho)
existe = os.path.exists(caminho)

# Jeito moderno (pathlib)
caminho = Path("/home") / "usuario" / "dados" / "arquivo.csv"
nome = caminho.name
existe = caminho.exists()
# Mais legível e orientado a objetos!
```
