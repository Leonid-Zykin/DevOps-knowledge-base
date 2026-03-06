## Module 09 — Mini-project: Full Infrastructure

### 1. Цель проекта

**Задача**: описать и развернуть инфраструктуру для приложения в облаке (или локально):

- VPC/сеть (или аналог);
- VM/EC2 (или несколько);
- S3/Storage для статики или бэкапов (если облако);
- Базовые security groups / firewall rules.

Оценочное время: **6–12 часов**.

---

### 2. Требования

1. **Структура**:
   - Разделение на модули: `modules/network`, `modules/compute`, `modules/storage` (или аналог).
   - Корневой `main.tf` вызывает модули и передаёт outputs между ними.
2. **Параметризация**:
   - Переменные для env (dev/prod), region, instance type.
   - tfvars для разных окружений.
3. **Outputs**:
   - ID VPC, публичные IP VM, URL S3 bucket (если применимо).
4. **State**:
   - Remote backend (S3/GCS/Azure) с locking — желательно.
   - Или локальный state для учебного проекта.

---

### 3. Пошаговый план

#### Шаг 1 — Модуль сети

- Создайте `modules/network` с VPC, subnets (public/private при необходимости).
- Используйте готовый модуль из registry или напишите свой.
- Outputs: vpc_id, subnet_ids, cidr_blocks.

#### Шаг 2 — Модуль compute

- Создайте `modules/compute` с EC2/VM.
- Используйте data source для AMI.
- Подключите к subnet из модуля network.
- Security group: SSH (22), HTTP (80) при необходимости.

#### Шаг 3 — Модуль storage (опционально)

- S3 bucket или аналог для статики/бэкапов.
- Настройте lifecycle или versioning при необходимости.

#### Шаг 4 — Root и переменные

- В корне: вызов модулей, передача outputs.
- variables.tf, outputs.tf.
- dev.tfvars, prod.tfvars.

#### Шаг 5 — Apply и проверка

```bash
terraform init
terraform plan -var-file=dev.tfvars
terraform apply -var-file=dev.tfvars
```

Проверьте доступность VM (SSH), работу сети.

#### Шаг 6 — Destroy

После проверки выполните `terraform destroy` и убедитесь, что ресурсы удалены.

---

### 4. Acceptance criteria

- [ ] Инфраструктура создаётся одной командой `terraform apply`
- [ ] Используются минимум 2 модуля
- [ ] Есть переменные и tfvars для разных окружений
- [ ] Outputs выводят ключевые идентификаторы
- [ ] `terraform destroy` корректно удаляет все ресурсы

---

### 5. Stretch goals

- Добавить Ansible playbook для конфигурации VM (подготовка к Module 10).
- Настроить remote backend с блокировкой.
- Добавить теги (Environment, Project, ManagedBy) ко всем ресурсам.

---

См. `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/modules]]`, `[[DevOps/06_Terraform/state-management]]`, `[[DevOps/06_Terraform/best-practices]]`.
