tags: [devops, terraform]

## Terraform Cheatsheet — команды и флаги

> ⚡ Tip: Всегда запускайте `terraform fmt` и `terraform validate` перед `plan/apply`, а состояние (`state`) держите в удалённом backend'е.

### Инициализация

```bash
terraform -version

terraform init                          # инициализация директории
terraform init -backend-config=backend.hcl
terraform init -upgrade                 # обновить провайдеры/модули
```

### Проверки и форматирование

```bash
terraform fmt                           # отформатировать *.tf
terraform fmt -recursive                # рекурсивно

terraform validate                      # проверить синтаксис и базовые ошибки
terraform validate -no-color
```

### План и применение

```bash
terraform plan                          # посмотреть diff
terraform plan -out=tfplan              # сохранить план в файл
terraform plan -var="env=stage" \
               -var-file=env/prod.tfvars

terraform apply                         # применить изменения (с подтверждением)
terraform apply -auto-approve           # без подтверждения (ОСТОРОЖНО)
terraform apply tfplan                  # применить ранее сохранённый план
```

### Destroy

```bash
terraform destroy                       # снести все управляемые ресурсы
terraform destroy -target=aws_s3_bucket.logs
terraform destroy -auto-approve
```

### State управление (локально/удалённо)

```bash
terraform state list                    # список ресурсов в state
terraform state show aws_s3_bucket.logs
terraform state rm aws_s3_bucket.logs   # удалить ресурс из state (не удаляя в облаке)
terraform state mv aws_instance.old aws_instance.new
terraform state pull > terraform.tfstate.backup
terraform state push terraform.tfstate  # вручную записать состояние (ОСТОРОЖНО)
```

Подробнее о состоянии: `[[state-management]]`.

### Targeting ресурсов

```bash
terraform plan -target=aws_s3_bucket.logs
terraform apply -target=module.network
```

Используйте только для временных/аварийных сценариев, не как обычный workflow.

### Workspaces

```bash
terraform workspace list
terraform workspace new stage
terraform workspace select stage
terraform workspace show
```

Workspaces = логические окружения поверх одного набора конфигов и одного backend'а.

### Import существующих ресурсов

```bash
terraform import aws_s3_bucket.logs my-logs-bucket
terraform import module.vpc.aws_vpc.this vpc-123456
```

> ⚡ Tip: Сначала создайте соответствующие ресурсы в `.tf`, затем делайте `import` и проверяйте `plan`.

### Логи и отладка

```bash
TF_LOG=INFO terraform plan
TF_LOG=DEBUG terraform apply
TF_LOG=TRACE TF_LOG_PATH=terraform.log terraform plan
```

### Модульные команды

```bash
terraform get                            # скачать модули
terraform get -update                    # обновить модули
```

Связанные заметки: `[[hcl-syntax]]`, `[[modules]]`, `[[state-management]]`, `[[best-practices]]`.

## Gotchas

- **Локальный `terraform.tfstate` в Git**  
  - **Проблема**: состояние с секретами и ресурсами попадает в репозиторий.  
  - **Решение**: добавлять `terraform.tfstate*` и `.terraform/` в `.gitignore`, использовать удалённый backend (S3/GCS/Azure Blob/etc).

- **`-auto-approve` в проде без diff**  
  - **Проблема**: необратимые изменения ресурсов без ревью (удаление БД, сетей и т.п.).  
  - **Решение**: в прод‑пайплайнах использовать отдельный шаг `plan` с ревью, затем apply только утверждённого плана.

- **Ручное редактирование `terraform.tfstate`**  
  - **Проблема**: легко сломать согласованность состояния и облака.  
  - **Решение**: для правок использовать `terraform state` команды, а не прямое редактирование.

> ⚠️ Warning: `terraform destroy` в каталоге с общими ресурсами (VPC, shared‑infra) может уничтожить критичные компоненты для множества окружений. Держите общую инфраструктуру и окружения в разных стейт‑файлах/модулях и ограничивайте, кто и где может делать `destroy`.

