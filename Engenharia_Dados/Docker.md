# 🐳 Docker

> Containerização de aplicações. Garante que o ambiente funciona igual em qualquer máquina.

**Relacionado:** [[Linux]] | [[Airflow]] | [[PostgreSQL]] | [[MongoDB]] | [[AWS]] | [[Terraform]] | [[Git_GitHub]]

---

## Conceitos
- **Imagem:** template read-only (como uma "receita")
- **Container:** instância em execução de uma imagem
- **Registry:** repositório de imagens (Docker Hub, ECR, etc.)
- **Dockerfile:** instruções para construir uma imagem
- **Volume:** dados persistentes fora do container
- **Network:** rede virtual entre containers

---

## Instalação (Ubuntu)
```bash
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Rodar sem sudo
sudo usermod -aG docker $USER && newgrp docker
```

---

## Comandos de Imagem
```bash
docker pull postgres:16             # baixar imagem
docker images                       # listar imagens locais
docker rmi postgres:16              # remover imagem
docker build -t minha-app:1.0 .     # construir imagem
docker push usuario/minha-app:1.0   # enviar para registry
docker tag local:tag remoto:tag     # criar tag alternativa
docker image prune                  # remover imagens não usadas
```

---

## Comandos de Container
```bash
# Criar e rodar
docker run postgres:16                    # em foreground
docker run -d postgres:16                 # background (detached)
docker run -d \
  --name meu-postgres \
  -e POSTGRES_PASSWORD=senha \
  -e POSTGRES_DB=meudb \
  -p 5432:5432 \                          # porta_host:porta_container
  -v postgres_data:/var/lib/postgresql/data \  # volume nomeado
  -v ./init.sql:/docker-entrypoint-initdb.d/init.sql \  # bind mount
  --restart unless-stopped \             # reiniciar automaticamente
  --memory 512m \                        # limitar memória
  --cpus 1 \                             # limitar CPU
  postgres:16

# Gerenciar
docker ps                                 # containers rodando
docker ps -a                              # todos (incluindo parados)
docker stop meu-postgres
docker start meu-postgres
docker restart meu-postgres
docker rm meu-postgres                    # remover parado
docker rm -f meu-postgres                 # forçar remoção
docker container prune                   # remover todos os parados

# Inspecionar
docker logs meu-postgres
docker logs -f meu-postgres              # follow (tempo real)
docker exec -it meu-postgres bash        # entrar no container
docker exec -it meu-postgres psql -U postgres
docker inspect meu-postgres             # JSON com todos os detalhes
docker stats                             # uso de recursos em tempo real
docker top meu-postgres                  # processos dentro do container
```

---

## Dockerfile
```dockerfile
FROM python:3.12-slim

LABEL maintainer="seu@email.com"

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Instalar dependências do sistema ANTES do código
# (melhor uso de cache de layer)
RUN apt-get update && apt-get install -y \
    gcc libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Dependências Python
COPY requirements.txt .
RUN pip install --upgrade pip && pip install -r requirements.txt

# Código da aplicação
COPY . .

# Usuário não-root (segurança)
RUN useradd --create-home appuser
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["python", "main.py"]
```

---

## Docker Compose
```yaml
# docker-compose.yml
version: '3.9'

services:
  postgres:
    image: postgres:16
    container_name: postgres
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: senha123
      POSTGRES_DB: pipeline
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - data-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  mongo:
    image: mongo:7
    container_name: mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: senha123
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    networks:
      - data-net

  app:
    build: .                          # constrói a partir do Dockerfile
    container_name: app
    depends_on:
      postgres:
        condition: service_healthy    # aguarda healthcheck passar
    environment:
      DATABASE_URL: postgresql://admin:senha123@postgres/pipeline
    volumes:
      - .:/app                        # bind mount para dev (live reload)
    ports:
      - "8000:8000"
    networks:
      - data-net

volumes:
  postgres_data:
  mongo_data:

networks:
  data-net:
    driver: bridge
```

```bash
# Comandos Docker Compose
docker compose up -d                    # sobe tudo em background
docker compose up -d postgres           # sobe só um serviço
docker compose down                     # derruba tudo
docker compose down -v                  # derruba + remove volumes
docker compose logs -f app              # logs de um serviço
docker compose ps                       # status
docker compose exec postgres bash       # entrar em um serviço
docker compose restart app              # reiniciar
docker compose build app                # rebuildar imagem
```

---

## Volumes vs Bind Mounts
```bash
# Volume nomeado (gerenciado pelo Docker — recomendado para produção)
-v meus_dados:/data

# Bind mount (pasta do host — ideal para desenvolvimento)
-v /home/usuario/projeto:/app
-v $(pwd):/app    # pasta atual
```
