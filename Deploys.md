# 🚀 Deploys — Colocar em Produção

> O código só tem valor quando está rodando. Estratégias e ferramentas para levar sua aplicação ao ar.

**Relacionado:** [[Docker]] | [[AWS]] | [[Azure]] | [[FastAPI]] | [[Airflow]] | [[Terraform]] | [[Git_GitHub]] | [[Streamlit]]

---

## Hierarquia de Complexidade de Deploy

```
1. Script local + cron                  → mais simples
2. VPS (EC2/VM) + Docker                → flex e barato
3. PaaS (Railway, Render, Heroku)       → sem gerenciar infra
4. Containers orquestrados (ECS, AKS)   → escala automática
5. Kubernetes (EKS, GKE, AKS)          → escala complexa → mais difícil
```

**Regra:** Use o nível mais simples que resolva o problema.

---

## 1. Dockerfile + VPS (Recomendado para começo)

```bash
# Na sua máquina — build e push para registry
docker build -t meu-usuario/minha-api:1.0.0 .
docker push meu-usuario/minha-api:1.0.0

# No servidor (EC2, DigitalOcean, etc.) — pull e run
ssh usuario@ip-do-servidor
docker pull meu-usuario/minha-api:1.0.0
docker run -d \
  --name minha-api \
  -p 80:8000 \
  -e DATABASE_URL="postgresql://..." \
  --restart unless-stopped \
  meu-usuario/minha-api:1.0.0
```

---

## 2. GitHub Actions — CI/CD Automatizado

```yaml
# .github/workflows/deploy.yml
name: Deploy para Produção

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: {python-version: "3.12"}
      - run: pip install -r requirements.txt
      - run: pytest --cov=src --cov-fail-under=80

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Login no Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build e Push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/minha-api:latest
            ${{ secrets.DOCKER_USERNAME }}/minha-api:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_USERNAME }}/minha-api:latest
            docker stop minha-api || true
            docker rm minha-api || true
            docker run -d \
              --name minha-api \
              -p 80:8000 \
              --env-file /opt/app/.env \
              --restart unless-stopped \
              ${{ secrets.DOCKER_USERNAME }}/minha-api:latest
            echo "✅ Deploy concluído!"
```

---

## 3. AWS ECS — Containers Gerenciados

```hcl
# Terraform para ECS Fargate
resource "aws_ecs_task_definition" "api" {
  family                   = "minha-api"
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = 256   # 0.25 vCPU
  memory                   = 512   # 512 MB

  container_definitions = jsonencode([{
    name  = "api"
    image = "meu-usuario/minha-api:latest"
    
    environment = [
      { name = "ENV", value = "prod" }
    ]
    
    secrets = [
      # Lê do Secrets Manager (nunca hardcode senha!)
      { name = "DATABASE_URL", valueFrom = aws_secretsmanager_secret.db_url.arn }
    ]
    
    portMappings = [{ containerPort = 8000 }]
    
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"  = "/ecs/minha-api"
        "awslogs-region" = "us-east-1"
      }
    }
  }])
}

resource "aws_ecs_service" "api" {
  name            = "minha-api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 2    # 2 instâncias
  launch_type     = "FARGATE"

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 8000
  }
}
```

---

## 4. Variáveis de Ambiente — Segurança em Produção

```bash
# .env (nunca commitar!)
DATABASE_URL=postgresql://admin:senha_secreta@db.host:5432/prod
SECRET_KEY=sua-chave-jwt-super-secreta
AWS_ACCESS_KEY_ID=AKIA...
SLACK_WEBHOOK_URL=https://hooks.slack.com/...

# Carregar no código
from dotenv import load_dotenv
import os

load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")
if not DATABASE_URL:
    raise ValueError("DATABASE_URL não configurada!")
```

```python
# Pydantic Settings (melhor para FastAPI)
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    workers: int = 4
    
    class Config:
        env_file = ".env"

settings = Settings()  # lê do .env automaticamente
```

---

## 5. Nginx — Proxy Reverso

```nginx
# /etc/nginx/sites-available/minha-api
server {
    listen 80;
    server_name api.meudominio.com;

    # Redirecionar para HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.meudominio.com;

    # Certificado SSL (Certbot / Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/api.meudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.meudominio.com/privkey.pem;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req zone=api burst=20 nodelay;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 60s;
        proxy_read_timeout 120s;
    }

    # Servir arquivos estáticos
    location /static/ {
        alias /var/www/static/;
        expires 30d;
    }
}
```

---

## 6. Estratégias de Deploy

### Blue-Green
```
Blue  (v1.0) → recebendo tráfego
Green (v2.0) → deploy da nova versão → testes
               ↓ tudo ok
Load Balancer → aponta para Green
Blue fica de stand-by (rollback instantâneo)
```

### Rolling
```
10 instâncias v1.0
→ substitui 2 por v2.0 → testa
→ substitui mais 2   → testa
→ ... até 10/10 em v2.0
Sem downtime, rollback gradual
```

### Canary
```
95% tráfego → v1.0
5% tráfego  → v2.0 (canary)
→ monitorar métricas/erros
→ se ok: aumentar para 10%, 25%, 50%, 100%
→ se não: 0% canary → rollback rápido
```

---

## Checklist de Deploy

```
Antes:
[ ] Testes automatizados passando (pytest)
[ ] Variáveis de ambiente configuradas
[ ] Backup do banco de dados
[ ] Migrations de banco revisadas

Durante:
[ ] Pipeline de CI/CD verde
[ ] Health check configurado
[ ] Logs visíveis (CloudWatch, Grafana)

Depois:
[ ] Monitorar métricas por 30 min
[ ] Verificar error rate
[ ] Comunicar time do sucesso
[ ] Atualizar CHANGELOG
```
