tags: [devops, terraform-hcl]

## Terraform HCL Syntax — переменные, locals, outputs, data, циклы

> ⚡ Tip: HCL читается как декларативный язык: **блоки** (`resource`, `module`, `variable`), **аргументы**, **выражения**. Используйте `terraform validate` и `terraform console` для проверки выражений.

### Переменные (`variable`)

```hcl
variable "env" {
  type        = string
  description = "Environment name"
  default     = "dev"
}

variable "allowed_ips" {
  type        = list(string)
  description = "CIDRs allowed to SSH"
  default     = ["10.0.0.0/8"]
}
```

Передача значений:

```bash
terraform plan -var="env=stage"
terraform plan -var-file=env/prod.tfvars
```

`prod.tfvars`:

```hcl
env         = "prod"
allowed_ips = ["1.2.3.4/32"]
```

### Locals

```hcl
locals {
  name_prefix = "${var.env}-${var.app_name}"
  common_tags = {
    Environment = var.env
    Terraform   = "true"
  }
}
```

Использование:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "${local.name_prefix}-logs"
  tags   = local.common_tags
}
```

### Outputs

```hcl
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "VPC ID"
}

output "app_url" {
  value = "https://${aws_lb.app.dns_name}"
}
```

Скрытие чувствительных значений:

```hcl
output "db_password" {
  value     = random_password.db.result
  sensitive = true
}
```

### Data sources

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

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

### Условия (тернарный оператор)

```hcl
resource "aws_instance" "web" {
  instance_type = var.env == "prod" ? "t3.medium" : "t3.micro"

  monitoring = var.enable_detailed_monitoring ? true : false
}
```

### Циклы: `count` и `for_each`

`count`:

```hcl
resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  tags = {
    Name = "web-${count.index}"
  }
}
```

`for_each`:

```hcl
variable "users" {
  type = set(string)
}

resource "aws_iam_user" "this" {
  for_each = var.users
  name     = each.value
}
```

### `for` выражения и `toset`/`tolist`

```hcl
locals {
  instance_names = ["api", "worker", "batch"]
  upper_names    = [for n in local.instance_names : upper(n)]

  tags_map = {
    for name in local.instance_names :
    name => "service-${name}"
  }
}
```

Фильтрация:

```hcl
locals {
  big_instances = [
    for k, inst in aws_instance.web :
    inst.id if inst.instance_type == "t3.large"
  ]
}
```

### Functions (часто используемые)

- `concat(list1, list2, ...)`
- `merge(map1, map2, ...)`
- `join(",", list)`
- `split(",", string)`
- `coalesce(v1, v2, ...)` — первый ненулевой/непустой;
- `length(list_or_map)`
- `lookup(map, key, default)`

Примеры:

```hcl
locals {
  all_tags = merge(
    var.default_tags,
    {
      Service = var.service_name
    }
  )
}
```

### terraform console

```bash
terraform console
> upper("dev")
> [for n in ["a","b","c"] : n if n != "b"]
> merge({a=1}, {b=2})
```

Связанные заметки: `[[terraform-cheatsheet]]`, `[[modules]]`, `[[best-practices]]`.

## Gotchas

- **Злоупотребление `count` с динамическими индексами**  
  - **Проблема**: при изменении количества ресурсов индексы смещаются, Terraform пытается пересоздать ресурсы.  
  - **Решение**: для идентифицируемых сущностей использовать `for_each` с ключами, а не `count.index`.

- **Жёстко зашитые строки вместо переменных/locals**  
  - **Проблема**: дублирование, сложно менять env/region.  
  - **Решение**: выносить повторяющиеся значения в `locals` и `variable`.

- **Секреты в plain‑text переменных/outputs**  
  - **Проблема**: чувствительные данные попадают в state, лог, outputs.  
  - **Решение**: помечать чувствительные outputs как `sensitive = true`, использовать внешние секрет‑хранилища (`[[10_Security/secrets-management]]`).

> ⚠️ Warning: Поскольку весь state Terraform хранит значения всех выражений и outputs, любые секреты, даже вычисленные внутри `locals` или `output`, оказываются в state‑файле. Рассматривайте state как чувствительный артефакт и защищайте его соответствующе.

