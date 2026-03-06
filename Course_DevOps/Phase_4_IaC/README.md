## Phase 4 — IaC (Infrastructure as Code)

**Недели 11–13** | Terraform + Ansible

В этой фазе вы научитесь описывать инфраструктуру кодом и настраивать серверы идемпотентно.

### Модули

| Модуль | Тема | Ссылки на базу знаний |
|--------|------|------------------------|
| **09** | Terraform | `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/hcl-syntax]]`, `[[DevOps/06_Terraform/modules]]`, `[[DevOps/06_Terraform/state-management]]`, `[[DevOps/06_Terraform/best-practices]]` |
| **10** | Ansible | `[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`, `[[DevOps/07_Ansible/inventory]]`, `[[DevOps/07_Ansible/jinja2-and-vars]]` |

### Результаты фазы

- Описываете инфраструктуру Terraform'ом (VPC, VM, S3 и т.п.) с модулями и remote state.
- Используете Ansible для конфигурации серверов и приложений.
- Понимаете, когда применять Terraform (инфраструктура), а когда Ansible (конфигурация).

### Порядок прохождения

1. **Module 09 — Terraform** — сначала инфраструктура.
2. **Module 10 — Ansible** — затем конфигурация созданных ресурсов.

### Облако

Для практики Terraform желательно иметь аккаунт в AWS, GCP или Yandex Cloud. Можно использовать локальные провайдеры (Docker, local) для базовых упражнений.
