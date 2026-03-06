## Module 09 — Terraform

### Почему это важно

Terraform — инструмент Infrastructure as Code (IaC). Вы описываете инфраструктуру декларативно, версионируете её в Git и применяете изменения предсказуемо. Без IaC инфраструктура превращается в «снежинки» и ручные изменения, которые сложно воспроизвести и откатить.

Опираемся на:
- `[[DevOps/06_Terraform/terraform-cheatsheet]]`
- `[[DevOps/06_Terraform/hcl-syntax]]`
- `[[DevOps/06_Terraform/modules]]`
- `[[DevOps/06_Terraform/state-management]]`
- `[[DevOps/06_Terraform/best-practices]]`

---

## 1. Основные концепции

- **Provider** — плагин для работы с облаком/сервисом (AWS, GCP, Azure, Docker, Kubernetes и т.д.).
- **Resource** — конкретный объект (VM, VPC, S3 bucket).
- **State** — файл/хранилище с текущим состоянием инфраструктуры; Terraform сравнивает его с желаемым и планирует изменения.
- **Plan** — предварительный просмотр изменений без применения.
- **Apply** — применение плана к реальной инфраструктуре.

---

## 2. Базовые команды

Из `[[DevOps/06_Terraform/terraform-cheatsheet]]`:

```bash
terraform init                    # инициализация, загрузка провайдеров
terraform fmt                     # форматирование *.tf
terraform validate                # проверка синтаксиса
terraform plan                    # план изменений
terraform apply                   # применить (с подтверждением)
terraform apply -auto-approve     # без подтверждения (осторожно!)
terraform destroy                 # удалить все управляемые ресурсы
```

> [!warning] Всегда запускайте `terraform fmt` и `terraform validate` перед plan/apply. State держите в удалённом backend.

---

## 3. Переменные, locals, outputs

Из `[[DevOps/06_Terraform/hcl-syntax]]`:

```hcl
variable "env" {
  type        = string
  description = "Environment"
  default     = "dev"
}

locals {
  name_prefix = "${var.env}-app"
  common_tags = {
    Environment = var.env
    Terraform   = "true"
  }
}

output "vpc_id" {
  value       = aws_vpc.main.id
  description = "VPC ID"
}
```

Передача: `-var="env=prod"`, `-var-file=prod.tfvars`.

---

## 4. Data sources

Чтение существующих ресурсов без управления ими:

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

---

## 5. Модули

Из `[[DevOps/06_Terraform/modules]]`:

```hcl
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  tags       = local.common_tags
}

resource "aws_subnet" "public" {
  vpc_id     = module.vpc.id
  cidr_block = "10.0.1.0/24"
}
```

Модули из registry:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  name    = "prod"
  cidr    = "10.0.0.0/16"
  # ...
}
```

---

## 6. State management

Из `[[DevOps/06_Terraform/state-management]]`:

- **Remote backend** — S3 + DynamoDB (AWS), GCS, Azure Blob — для команды и блокировок.
- **Locking** — предотвращает параллельный apply.
- **Workspaces** — логические окружения (dev/stage/prod) поверх одного backend.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
  }
}
```

> [!warning] State содержит чувствительные данные. Не коммитьте его в Git. Используйте remote backend с шифрованием.

---

## 7. Best practices

Из `[[DevOps/06_Terraform/best-practices]]`:

- Делите Terraform по доменам и окружениям.
- Используйте модули для переиспользования.
- Стандартизируйте теги (Environment, ManagedBy, Project).
- Запускайте Terraform из CI, не с локальных машин.
- Не используйте `-target` в обычном workflow.

---

## Что Middle DevOps должен знать по Terraform (чек‑лист)

- [ ] Умею инициализировать проект, plan, apply, destroy (`[[DevOps/06_Terraform/terraform-cheatsheet]]`).
- [ ] Понимаю переменные, locals, outputs и data sources (`[[DevOps/06_Terraform/hcl-syntax]]`).
- [ ] Могу создать и использовать модуль (локальный и из registry) (`[[DevOps/06_Terraform/modules]]`).
- [ ] Понимаю remote state и locking (`[[DevOps/06_Terraform/state-management]]`).
- [ ] Следую best practices: теги, структура, CI (`[[DevOps/06_Terraform/best-practices]]`).
