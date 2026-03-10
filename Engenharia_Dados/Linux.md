# 🐧 Linux

> Sistema operacional dominante em servidores e ambientes de dados. Dominar o terminal é essencial.

**Relacionado:** [[Windows]] | [[Docker]] | [[Git_GitHub]] | [[Python]] | [[Ambientes_Virtuais]] | [[pyenv]]

---

## Navegação e Arquivos
```bash
# Localização
pwd                        # onde estou
ls                         # listar arquivos
ls -la                     # detalhes + ocultos
ls -lh                     # tamanhos legíveis (KB, MB, GB)
ls -lt                     # ordenado por data
tree                       # estrutura em árvore (instalar: apt install tree)

# Navegar
cd ~/Desktop               # ir para Desktop
cd ..                      # subir um nível
cd -                       # voltar ao diretório anterior
cd ~                       # ir para home

# Criar
mkdir nova_pasta
mkdir -p caminho/longo/aninhado   # cria toda a estrutura
touch arquivo.txt                  # cria arquivo vazio

# Copiar / Mover
cp origem.txt destino.txt
cp -r pasta_origem/ pasta_destino/ # copiar pasta (recursivo)
mv arquivo.txt novo_nome.txt       # renomear
mv arquivo.txt /outra/pasta/       # mover

# Remover (CUIDADO — sem lixeira!)
rm arquivo.txt
rm -rf pasta/               # recursivo e forçado
rmdir pasta_vazia/
```

---

## Visualizar Arquivos
```bash
cat arquivo.txt              # exibe tudo
head -20 arquivo.txt         # primeiras 20 linhas
tail -20 arquivo.txt         # últimas 20 linhas
tail -f arquivo.log          # monitorar em tempo real
less arquivo.txt             # paginar (q para sair)
wc -l arquivo.txt            # contar linhas
```

---

## Busca
```bash
# Buscar arquivos
find . -name "*.csv"
find . -type f -size +100M              # arquivos > 100MB
find . -mtime -7                        # modificados nos últimos 7 dias
find . -name "*.py" -not -path "./.venv/*"  # excluir pasta

# Buscar conteúdo
grep "erro" arquivo.log
grep -r "conexao" .                     # recursivo
grep -n "def " script.py               # mostra número da linha
grep -i "ERROR" logs/*.log              # case insensitive
grep -v "DEBUG" app.log                 # inverso (linhas SEM DEBUG)
grep -E "ERROR|WARNING" app.log         # regex

# Busca avançada: ripgrep (instalar: apt install ripgrep)
rg "conexao" --type py                  # mais rápido que grep
```

---

## Permissões
```bash
# Ver permissões
ls -l arquivo.txt
# -rw-r--r-- 1 usuario grupo 1234 Jan 01 arquivo.txt
# [tipo][dono][grupo][outros]
# tipo: - arquivo, d diretório, l link simbólico
# rwx = read, write, execute

# Alterar permissões
chmod 755 script.sh        # rwxr-xr-x
chmod 644 arquivo.txt      # rw-r--r--
chmod +x script.sh         # adicionar permissão de execução
chmod -R 755 pasta/        # recursivo

# Alterar dono
chown usuario:grupo arquivo.txt
sudo chown -R www-data:www-data /var/www/
```

---

## Processos e Sistema
```bash
top                         # monitor de processos
htop                        # versão melhorada (apt install htop)
ps aux | grep python        # processos do Python
kill 1234                   # encerrar processo pelo PID
kill -9 1234                # forçar encerramento
pkill python                # encerrar por nome

# Recursos do sistema
df -h                       # uso do disco
du -sh pasta/               # tamanho de uma pasta
du -sh * | sort -h          # ordenar por tamanho
free -h                     # memória
nproc                       # número de CPUs
uname -a                    # info do sistema

# Variáveis de ambiente
export MINHA_VAR="valor"
echo $MINHA_VAR
env                         # listar todas
printenv PATH

# Adicionar ao PATH
export PATH="$HOME/.local/bin:$PATH"
# Para persistir, adicionar ao ~/.bashrc
```

---

## Gerenciador de Pacotes (apt)
```bash
sudo apt update                        # atualizar lista
sudo apt upgrade                       # atualizar pacotes
sudo apt install nome-pacote
sudo apt remove nome-pacote
sudo apt purge nome-pacote             # remove + config
sudo apt autoremove                    # limpar órfãos
apt search python                      # buscar pacote
apt list --installed | grep python     # listar instalados
```

---

## Redirecionamento e Pipes
```bash
# Redirecionar saída
comando > saida.txt          # sobreescreve
comando >> saida.txt         # acrescenta
comando 2> erros.txt         # só erros (stderr)
comando > tudo.txt 2>&1      # stdout + stderr juntos
comando > /dev/null          # descartar saída

# Pipe — encadear comandos
cat dados.csv | grep "SP" | wc -l
ls -la | sort -k5 -n          # ordenar por tamanho
ps aux | grep python | grep -v grep
cat log.txt | tail -100 | grep ERROR | head -20
```

---

## Atalhos do Terminal
| Atalho | Ação |
|--------|------|
| `Ctrl+C` | Cancela processo |
| `Ctrl+Z` | Suspende processo |
| `Ctrl+D` | Fecha terminal / EOF |
| `Ctrl+L` | Limpa tela |
| `Ctrl+A` | Início da linha |
| `Ctrl+E` | Fim da linha |
| `Ctrl+R` | Busca no histórico |
| `Ctrl+U` | Apaga até o início |
| `Ctrl+K` | Apaga até o fim |
| `Tab` | Autocomplete |
| `!!` | Repete último comando |
| `sudo !!` | Repete com sudo |
| `Alt+.` | Último argumento do comando anterior |

---

## Shell Script
```bash
#!/bin/bash
set -euo pipefail    # para em erro, trata undefined vars, pipes

AMBIENTE="${1:-dev}"  # argumento ou "dev" como padrão
DATA=$(date +%Y-%m-%d)
LOG_FILE="logs/pipeline_${DATA}.log"

mkdir -p logs

log() { echo "[$(date '+%H:%M:%S')] $1" | tee -a "$LOG_FILE"; }

log "Iniciando pipeline - Ambiente: $AMBIENTE"

if [ "$AMBIENTE" = "prod" ]; then
    log "Rodando em produção!"
fi

for arquivo in data/raw/*.csv; do
    log "Processando: $arquivo"
    python processar.py "$arquivo" >> "$LOG_FILE" 2>&1
done

log "Pipeline finalizado"
```
