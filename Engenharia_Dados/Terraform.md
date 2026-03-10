# 🏗️ Terraform

> Infrastructure as Code (IaC). Define e provisiona infraestrutura usando código versionável.

**Relacionado:** [[AWS]] | [[Docker]] | [[Git_GitHub]] | [[Linux]]

---

## Conceitos
- **Provider:** plugin que se comunica com APIs (AWS, GCP, Azure, etc.)
- **Resource:** componente de infraestrutura a ser criado
- **Data Source:** consulta recursos existentes sem criá-los
- **State:** arquivo que mapeia recursos reais ↔ código (`terraform.tfstate`)
- **Module:** conjunto reutilizável de recursos
- **Variable:** parâmetros de entrada
- **Output:** valores de saída do módulo/configuração

---

## Instalação
```bash
# Ubuntu
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

terraform version
```

---

## Estrutura de Projeto
```
terraform/
├── main.tf           # recursos principais
├── variables.tf      # declaração de variáveis
├── outputs.tf        # valores de saída
├── terraform.tfvars  # valores (não commitar se tiver segredos!)
├── versions.tf       # versão do Terraform e providers
└── modules/
    ├── rds/
    │   ├── main.tf
    │   └── variables.tf
    └── s3/
        └── main.tf
```

---

## Configuração para AWS
```hcl
# versions.tf
terraform {
  required_version = ">= 1.6"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # Backend remoto (recomendado para times)
  backend "s3" {
    bucket = "meu-terraform-state"
    key    = "pipeline/terraform.tfstate"
    region = "us-east-1"
    
    # Lock para evitar conflitos simultâneos
    dynamodb_table = "terraform-state-lock"
  }
}

provider "aws" {
  region = var.aws_region
}


# main.tf
resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.projeto}-data-lake-${var.ambiente}"

  tags = {
    Projeto   = var.projeto
    Ambiente  = var.ambiente
    ManagedBy = "Terraform"
  }
}

resource "aws_s3_bucket_versioning" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  rule {
    id     = "mover-para-glacier"
    status = "Enabled"
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
  }
}

# RDS PostgreSQL
resource "aws_db_instance" "postgres" {
  identifier        = "${var.projeto}-${var.ambiente}"
  engine            = "postgres"
  engine_version    = "16"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  
  db_name  = var.db_name
  username = var.db_user
  password = var.db_password  # use Secrets Manager em produção
  
  skip_final_snapshot = var.ambiente != "prod"
  
  tags = { Ambiente = var.ambiente }
}


# variables.tf
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "projeto" {
  type = string
}

variable "ambiente" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.ambiente)
    error_message = "Ambiente deve ser dev, staging ou prod."
  }
}

variable "db_password" {
  type      = string
  sensitive = true   # não aparece nos logs
}


# outputs.tf
output "data_lake_bucket" {
  value = aws_s3_bucket.data_lake.bucket
}

output "rds_endpoint" {
  value     = aws_db_instance.postgres.endpoint
  sensitive = true
}
```

---

## Comandos Principais
```bash
terraform init          # inicializa providers e backend
terraform plan          # preview das mudanças (diff)
terraform apply         # aplica as mudanças
terraform apply -auto-approve   # sem confirmação manual
terraform destroy       # destrói TODA a infraestrutura
terraform fmt           # formata os arquivos .tf
terraform validate      # valida a sintaxe

# State
terraform show          # ver estado atual
terraform state list    # listar recursos no state
terraform output        # ver outputs
terraform output rds_endpoint

# Importar recursos existentes
terraform import aws_s3_bucket.meu meu-bucket-name

# Targets — aplicar só um recurso
terraform apply -target=aws_s3_bucket.data_lake
```

---

## terraform.tfvars
```hcl
# terraform.tfvars (NÃO commitar se tiver senhas!)
aws_region  = "us-east-1"
projeto     = "data-pipeline"
ambiente    = "dev"
db_password = "minha-senha-segura"
```
