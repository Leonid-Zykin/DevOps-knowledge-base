## Module 10 — Mini-project: Configure Servers After Terraform

### 1. Цель проекта

**Задача**: написать Ansible playbook/roles для конфигурации серверов, созданных Terraform (из Module 09):

- базовая настройка (пакеты, timezone, пользователи);
- установка и настройка веб-сервера или приложения;
- подготовка к деплою (Docker, systemd и т.п.).

Оценочное время: **4–8 часов**.

---

### 2. Требования

1. **Inventory**:
   - Статический или dynamic (из Terraform outputs / AWS EC2).
   - Группы: web, db (если применимо), common для всех.
2. **Roles**:
   - `common` — базовые пакеты, timezone, SSH hardening (опционально).
   - `web` — nginx/apache или ваше приложение.
   - Опционально: `db` для настройки PostgreSQL/Redis.
3. **Переменные**:
   - group_vars для окружений (dev/prod).
   - Использование Jinja2 в шаблонах.
4. **Секреты**:
   - Ansible Vault для паролей БД, API-ключей.

---

### 3. Пошаговый план

#### Шаг 1 — Inventory из Terraform

- Экспортируйте IP хостов из Terraform output.
- Создайте inventory.ini или скрипт для генерации inventory из `terraform output -json`.
- Альтернатива: статический inventory с IP из Terraform.

#### Шаг 2 — Role common

- Установка пакетов (curl, git, htop, vim).
- Настройка timezone.
- Опционально: создание пользователя, настройка sudo.

#### Шаг 3 — Role web

- Установка nginx (или аналога).
- Шаблон конфига с переменными (port, server_name).
- Handler для перезапуска при изменении конфига.

#### Шаг 4 — Интеграция с Terraform

- После `terraform apply` получить IP и запустить Ansible.
- Пример: `terraform output -raw web_ips | tr ',' '\n'` и подставить в inventory.

#### Шаг 5 — Vault для секретов

- Создать vault.yml с паролями.
- Использовать в шаблонах через `{{ vault_* }}`.
- Документировать запуск с `--ask-vault-pass`.

---

### 4. Acceptance criteria

- [ ] Playbook применяется к хостам из Terraform
- [ ] Используются минимум 2 роли (common + web или аналог)
- [ ] Переменные вынесены в group_vars
- [ ] Секреты в Vault (если есть чувствительные данные)
- [ ] Повторный запуск идемпотентен (changed=0 при отсутствии изменений)

---

### 5. Stretch goals

- Dynamic inventory для AWS EC2 (плагин amazon.aws.aws_ec2).
- Роль для установки Docker и запуска контейнеров.
- Отдельные playbooks для dev и prod с разными group_vars.

---

См. `[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`, `[[DevOps/07_Ansible/inventory]]`, `[[DevOps/06_Terraform/terraform-cheatsheet]]`.
