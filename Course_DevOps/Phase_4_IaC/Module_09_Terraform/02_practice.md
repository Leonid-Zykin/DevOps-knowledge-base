## Module 09 — Terraform Practice

Terraform CLI установлен. Для облачных провайдеров нужны credentials (AWS CLI, gcloud, az и т.д.). Можно использовать провайдер `local` или `docker` для локальной практики без облака.

Опирайтесь на `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/hcl-syntax]]`, `[[DevOps/06_Terraform/modules]]`.

---

### Task 1 — Инициализация и первый ресурс

- **Goal**: Создать проект и применить простой ресурс.
- **Steps**:
  1. Создайте директорию `tf-basics`, перейдите в неё.
  2. Создайте `main.tf` с provider (например, `local` или `aws`) и одним ресурсом (local_file или aws_s3_bucket).
  3. Выполните: `terraform init`, `terraform plan`, `terraform apply`.
  4. Проверьте, что ресурс создан.
- **Expected result**: Ресурс создан, state содержит запись.

---

### Task 2 — Переменные и tfvars

- **Goal**: Параметризовать конфигурацию.
- **Steps**:
  1. Добавьте `variables.tf` с переменными (env, name_prefix).
  2. Создайте `dev.tfvars` и `prod.tfvars` с разными значениями.
  3. Запустите `terraform plan -var-file=dev.tfvars` и `terraform plan -var-file=prod.tfvars`.
  4. Убедитесь, что план меняется в зависимости от файла.
- **Expected result**: Одна конфигурация, разные окружения через tfvars.

---

### Task 3 — Outputs

- **Goal**: Экспортировать значения для других систем.
- **Steps**:
  1. Добавьте `outputs.tf` с output (например, id ресурса, URL).
  2. После apply выполните `terraform output`.
  3. Используйте output в другом ресурсе через `resource.attr` (если применимо).
- **Expected result**: Outputs отображаются и могут использоваться в коде.

---

### Task 4 — Data source

- **Goal**: Использовать существующие данные.
- **Steps**:
  1. Добавьте data source (например, `aws_ami`, `aws_availability_zones` или аналог для вашего провайдера).
  2. Используйте результат data в resource (ami, zone и т.п.).
  3. Выполните plan/apply.
- **Expected result**: Ресурс создаётся с данными из data source.

---

### Task 5 — Локальный модуль

- **Goal**: Вынести логику в модуль.
- **Steps**:
  1. Создайте `modules/network` с variables.tf, main.tf, outputs.tf.
  2. В модуле опишите ресурс (VPC, subnet или local_file с конфигом).
  3. В корне вызовите `module "network" { source = "./modules/network" ... }`.
  4. Выполните `terraform init`, plan, apply.
- **Expected result**: Модуль применяется, outputs доступны через `module.network.attr`.

---

### Task 6 — Модуль из registry

- **Goal**: Использовать готовый модуль.
- **Steps**:
  1. Найдите модуль в Terraform Registry (например, terraform-aws-modules/vpc/aws).
  2. Добавьте `module "vpc" { source = "..."; version = "..." }` с нужными параметрами.
  3. Выполните `terraform init -upgrade`, plan, apply (если есть облако).
- **Expected result**: Инфраструктура создана через сторонний модуль.

---

### Task 7 — Remote backend (опционально)

- **Goal**: Настроить удалённый state.
- **Steps**:
  1. Создайте S3 bucket и DynamoDB table (или аналог для GCP/Azure).
  2. Добавьте блок `terraform { backend "s3" { ... } }`.
  3. Выполните `terraform init -migrate-state` (если был локальный state).
  4. Проверьте, что state в облаке.
- **Expected result**: State хранится удалённо, локальный файл отсутствует.

---

### Task 8 — terraform destroy

- **Goal**: Безопасно удалить ресурсы.
- **Steps**:
  1. Выполните `terraform plan -destroy` и изучите план.
  2. Выполните `terraform destroy` (подтвердите или используйте -auto-approve в тестовой среде).
  3. Убедитесь, что ресурсы удалены.
- **Expected result**: Вся управляемая инфраструктура удалена.

---

### Task 9 — Break it: изменить state вручную

- **Goal**: Понять важность state.
- **Steps**:
  1. Создайте ресурс через Terraform.
  2. Удалите или измените ресурс вручную (в облаке/файловой системе).
  3. Запустите `terraform plan` — Terraform предложит recreate или update.
  4. Разберитесь, как синхронизировать state (import, корректировка).
- **Expected result**: Понимание drift и способов его устранения.

---

### Task 10 — Break it: конфликт при параллельном apply

- **Goal**: Понять необходимость locking.
- **Steps**:
  1. В двух терминалах одновременно запустите `terraform apply` (без remote backend с locking).
  2. Наблюдайте возможные конфликты или дублирование ресурсов.
  3. Обсудите, почему remote backend с DynamoDB/GCS locking решает проблему.
- **Expected result**: Понимание, зачем нужен locking в state.

---

## Итог

После практики вы умеете:
- создавать и применять Terraform-конфигурации;
- использовать переменные, outputs, data sources;
- работать с модулями (локальными и из registry);
- понимать state и remote backend.
