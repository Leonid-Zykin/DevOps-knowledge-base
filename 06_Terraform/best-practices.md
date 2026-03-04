tags: [devops, terraform-best-practices]

## Terraform Best Practices — структура, нейминг, DRY

> ⚡ Tip: Делите Terraform по доменам и окружениям, максимально упрощайте root‑модули и стандартизируйте теги, именование и backends.

### Структура проекта

Вариант 1 (по окружениям):

```text
envs/
  dev/
    main.tf
    variables.tf
    backend.tf
  stage/
  prod/
modules/
  vpc/
  eks/
  app/
```

Вариант 2 (monorepo + workspace):

```text
main.tf
variables.tf
backend.tf
modules/
  ...
```

И `terraform.workspace` в backend key.

### Нейминг и теги

Используйте `locals`:

```hcl
locals {
  name_prefix = "${var.project}-${var.env}"
  common_tags = {
    Project     = var.project
    Environment = var.env
    ManagedBy   = "Terraform"
  }
}
```

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "${local.name_prefix}-logs"
  tags   = local.common_tags
}
```

### Разделение ответственности

- **core**: VPC, IAM, shared‑сервисы;
- **platform**: кластеры (EKS/GKE/AKS), мониторинг, CI;
- **apps**: отдельные приложения/сервисы.

Каждый слой — свой state и свои модули.

### DRY через модули

- повторяющиеся паттерны (VPC, ALB, RDS, S3, CloudFront) выносить в модули;
- root‑модули должны быть thin: в основном вызовы модулей + variables.

### Работа через CI/CD

- Terraform запускается только из CI;
- `plan` и `apply` разделены:
  - `plan` → артефакт + diff для ревью;
  - `apply` → только подтверждённый план.

Связанные: `[[05_CI-CD/concepts]]`, `[[06_Terraform/state-management]]`, `[[06_Terraform/modules]]`.

## Gotchas

- **Смешивание prod/dev в одном state и конфиге**  
  - **Проблема**: легко случайно задеть prod при изменении dev.  
  - **Решение**: отдельные backend key/каталоги для окружений, разные доступы.

- **Отсутствие единой схемы тегирования**  
  - **Проблема**: неудобный поиск и учёт ресурсов, сложности с биллингом.  
  - **Решение**: централизованный набор тегов (Project, Env, Owner, CostCenter) и проверка их наличия (policy as code).

- **Сложные многоуровневые модули без документации**  
  - **Проблема**: через год никто не понимает, как работает модуль.  
  - **Решение**: документация (README, примеры), минимизация «магии» внутри модулей, семантическое версионирование.

> ⚠️ Warning: Любые крупные рефакторинги структуры Terraform (разделение стейтов, переименование модулей, перенос ресурсов) нужно планировать и тестировать на отдельных окружениях. Ошибки на этом уровне могут приводить к массовому recreate ресурсов (включая БД и сети).

