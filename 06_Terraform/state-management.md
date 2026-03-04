tags: [devops, terraform-state]

## Terraform State Management — remote state, locking, workspaces, import

> ⚡ Tip: State = источник правды о созданных ресурсах. Держите его в удалённом backend'е с блокировками (S3+DynamoDB, GCS, Azure Blob, etc.).

### Локальный vs удалённый state

По умолчанию:

- `terraform.tfstate` хранится локально в каталоге.

Удалённый backend (`backend "s3"` пример):

```hcl
terraform {
  backend "s3" {
    bucket         = "tf-state-bucket"
    key            = "project/prod/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "tf-state-locks"
    encrypt        = true
  }
}
```

Инициализация/миграция:

```bash
terraform init
terraform init -migrate-state
```

### Просмотр и правка state

```bash
terraform state list                    # все ресурсы в стейте
terraform state show aws_s3_bucket.logs # детали ресурса

terraform state pull > terraform.tfstate.backup
terraform state push terraform.tfstate  # загрузить изменённый стейт (ОСТОРОЖНО)
```

Перемещение и удаление:

```bash
terraform state mv aws_instance.old aws_instance.new
terraform state rm aws_s3_bucket.logs
```

### Import существующих ресурсов

```bash
terraform import aws_s3_bucket.logs my-logs-bucket
terraform import module.vpc.aws_vpc.this vpc-123456
```

Шаги:

1. описать ресурс в `.tf`;
2. выполнить `terraform import`;
3. проверить `terraform plan`.

### Locking

- S3 + DynamoDB: `dynamodb_table` для блокировок;
- GCS: встроенный locking;
- локальный state: блокировку обеспечивает локальная файловая система.

При конфликте:

```bash
Error acquiring the state lock
```

- дождаться завершения другого `plan/apply`;
- если lock «завис» — удалить запись в DynamoDB/хранилище (с осторожностью).

### Workspaces и разделение окружений

```bash
terraform workspace list
terraform workspace new stage
terraform workspace select stage

terraform workspace show
```

В backend'е key обычно включает workspace:

```hcl
key = "project/${terraform.workspace}/terraform.tfstate"
```

Альтернатива: отдельные каталоги/репозитории для окружений (`env/prod`, `env/stage`).

Связанные: `[[terraform-cheatsheet]]`, `[[modules]]`, `[[best-practices]]`.

## Gotchas

- **Один общий state для всего**  
  - **Проблема**: один `terraform destroy` или ошибка в конфиге может снести пол‑инфраструктуры.  
  - **Решение**: делить state по доменам (network/core/app/env), использовать разные backend key/каталоги.

- **Отсутствие locking в multi‑user сценариях**  
  - **Проблема**: одновременный `plan/apply` приводит к гонкам и порче state.  
  - **Решение**: использовать backend с блокировками (S3+DynamoDB, GCS, Azure) и запускать Terraform только из CI/CD.

- **Ручной `state push` без полного понимания**  
  - **Проблема**: несогласованность состояния и реальных ресурсов, потеря ссылок.  
  - **Решение**: правки state делать только через `terraform state` команды, raw‑редактирование — как крайний случай с резервной копией.

> ⚠️ Warning: State‑файл содержит все атрибуты ресурсов, включая чувствительные (пароли, ключи, токены, строки подключения). Его компрометация = компрометация всей инфраструктуры. Храните state только в защищённых хранилищах, с шифрованием, бэкапами и ограниченным доступом.

