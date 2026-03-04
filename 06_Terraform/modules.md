tags: [devops, terraform-modules]

## Terraform Modules — структура, registry, версионирование

> ⚡ Tip: Модули помогают переиспользовать инфраструктурные шаблоны (VPC, кластеры, сервисы) и навязать единые стандарты тегирования, именования и безопасности.

### Структура локального модуля

```text
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
```

`modules/vpc/variables.tf`:

```hcl
variable "cidr_block" {
  type = string
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

`modules/vpc/main.tf`:

```hcl
resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags                 = var.tags
}
```

`modules/vpc/outputs.tf`:

```hcl
output "id" {
  value = aws_vpc.this.id
}
```

### Использование локального модуля

```hcl
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
  tags = {
    Name        = "prod-vpc"
    Environment = "prod"
  }
}

resource "aws_subnet" "public" {
  vpc_id     = module.vpc.id
  cidr_block = "10.0.1.0/24"
}
```

### Модули из registry

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "prod"
  cidr = "10.0.0.0/16"

  azs             = ["eu-central-1a", "eu-central-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.3.0/24", "10.0.4.0/24"]
}
```

> ⚡ Tip: Всегда фиксируйте версию модуля (`version` или `ref=tag` для Git‑источников).

### Источники модулей (source)

```hcl
# относительный путь
source = "./modules/network"

# Git
source = "git::https://github.com/org/tf-modules.git//vpc?ref=v1.2.3"

# Git (SSH)
source = "git@github.com:org/tf-modules.git//eks?ref=main"
```

### version constraints

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"     # фиксированная версия
}

# semver
version = "~> 5.1"      # >=5.1.0, <6.0.0
version = ">= 5.0, < 6.0"
```

### Паттерн root‑модуля

Корень репо = root‑модуль, который использует другие модули:

```hcl
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = var.vpc_cidr
  tags       = local.common_tags
}

module "eks" {
  source  = "./modules/eks"
  vpc_id  = module.vpc.id
  subnets = module.vpc.private_subnet_ids
}
```

Связанные заметки: `[[hcl-syntax]]`, `[[terraform-cheatsheet]]`, `[[best-practices]]`.

## Gotchas

- **Модули без версии (latest)**  
  - **Проблема**: незаметные изменения модулей ломают инфраструктуру.  
  - **Решение**: всегда указывать версию, обновлять осознанно и постепенно.

- **Смешивание разных типов окружений в одном root‑модуле**  
  - **Проблема**: сложно управлять prod/dev конфигами, высок риск deploy в неправильное окружение.  
  - **Решение**: отделять окружения по каталогам или workspaces, чётко разделять стейт (см. `[[state-management]]`).

- **Жёстко зашитые region/account в модулях**  
  - **Проблема**: модуль сложно переиспользовать в другом регионе/аккаунте.  
  - **Решение**: параметризовать все привязки через переменные или `provider` конфигурацию.

> ⚠️ Warning: Переиспользуемые модули — это фактически внутренняя платформа. Ошибка/баг в модуле, используемом десятками проектов, может «тихо» сломать инфраструктуру при следующем `plan/apply`. Используйте версионирование, changelog и постепенные rollout'ы.

