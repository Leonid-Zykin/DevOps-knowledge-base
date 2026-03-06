## Module 09 — Terraform Quiz

Ориентируйтесь на `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/hcl-syntax]]`, `[[DevOps/06_Terraform/modules]]`, `[[DevOps/06_Terraform/state-management]]`.

---

### Вопрос 1 — terraform plan

Что делает `terraform plan`?

a) Применяет изменения к инфраструктуре  
b) Показывает планируемые изменения без применения  
c) Удаляет все ресурсы  
d) Инициализирует провайдеры

---

### Вопрос 2 — State

Для чего Terraform хранит state?

a) Только для отладки  
b) Чтобы сравнивать желаемое состояние с фактическим и планировать изменения  
c) Для кэширования провайдеров  
d) Для хранения переменных

---

### Вопрос 3 — Data source

Чем data source отличается от resource?

a) Ничем  
b) Data source читает существующие ресурсы, не управляет ими  
c) Data source создаёт ресурсы быстрее  
d) Resource нельзя использовать с data source

---

### Вопрос 4 — Модуль

Как вызвать локальный модуль?

a) `import "./modules/vpc"`  
b) `module "vpc" { source = "./modules/vpc" }`  
c) `include "vpc"`  
d) `require "./modules/vpc"`

---

### Вопрос 5 — Переменные

Как передать переменную при plan?

a) Только через файл  
b) `-var="name=value"` или `-var-file=file.tfvars`  
c) Только через environment variables  
d) Переменные нельзя передавать

---

### Вопрос 6 — Output

Как использовать output одного модуля в другом ресурсе?

a) Через `output.module_name.attr`  
b) Через `module.module_name.output_name`  
c) Через `var.module_name_output`  
d) Output нельзя использовать в коде

---

### Вопрос 7 — Remote backend

Зачем нужен remote backend (S3, GCS)?

a) Чтобы быстрее загружать провайдеры  
b) Для совместной работы команды, блокировок и хранения state в одном месте  
c) Чтобы не платить за локальный диск  
d) Remote backend не нужен

---

### Вопрос 8 — terraform destroy

Что делает `terraform destroy`?

a) Удаляет только state  
b) Удаляет все ресурсы, управляемые Terraform в текущем state  
c) Откатывает последний apply  
d) Удаляет провайдеры

---

### Вопрос 9 — Locking

Зачем нужен locking в state backend?

a) Чтобы шифровать state  
b) Чтобы предотвратить параллельный apply и повреждение state  
c) Чтобы ограничить доступ по IP  
d) Locking не используется в Terraform

---

### Вопрос 10 — Best practice

Почему не рекомендуется запускать Terraform apply с локальной машины разработчика в проде?

a) Terraform работает только в CI  
b) Риск потери state, отсутствия аудита, рассинхронизации с командой  
c) Локальная машина всегда медленнее  
d) Это не best practice

---

> [!note]- Answers
> **1** — b) Показывает планируемые изменения без применения.  
> **2** — b) Сравнение желаемого с фактическим, планирование изменений.  
> **3** — b) Data source читает, не управляет.  
> **4** — b) `module "vpc" { source = "./modules/vpc" }`.  
> **5** — b) `-var="name=value"` или `-var-file=file.tfvars`.  
> **6** — b) `module.module_name.output_name`.  
> **7** — b) Совместная работа, блокировки, единое хранилище.  
> **8** — b) Удаляет все управляемые ресурсы.  
> **9** — b) Предотвращение параллельного apply.  
> **10** — b) Риск потери state, отсутствие аудита, рассинхронизация.
