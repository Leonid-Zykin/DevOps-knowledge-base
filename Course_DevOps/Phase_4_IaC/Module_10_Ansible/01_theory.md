## Module 10 — Ansible

### Почему это важно

Ansible — инструмент для идемпотентной конфигурации серверов и оркестрации. В отличие от Terraform (создание инфраструктуры), Ansible настраивает уже созданные хосты: пакеты, конфиги, сервисы, пользователи. В связке Terraform + Ansible вы получаете полный цикл: инфраструктура → конфигурация.

Опираемся на:
- `[[DevOps/07_Ansible/ansible-cheatsheet]]`
- `[[DevOps/07_Ansible/playbooks-and-roles]]`
- `[[DevOps/07_Ansible/inventory]]`
- `[[DevOps/07_Ansible/jinja2-and-vars]]`

---

## 1. Основные концепции

- **Inventory** — список хостов и групп; определяет, куда применять playbook.
- **Playbook** — YAML-файл с plays и tasks.
- **Task** — единица работы (установить пакет, скопировать файл, запустить сервис).
- **Role** — переиспользуемый набор tasks, handlers, templates, vars.
- **Idempotency** — повторный запуск не должен менять уже настроенное состояние.

---

## 2. Базовые команды

Из `[[DevOps/07_Ansible/ansible-cheatsheet]]`:

```bash
ansible all -m ping -i inventory.ini
ansible web -i inventory.ini -m apt -a "name=nginx state=present" -b
ansible-playbook site.yml -i inventory.ini
ansible-playbook site.yml --check --diff    # dry-run
ansible-playbook site.yml --tags packages   # только tasks с tag
ansible-playbook site.yml --limit web1      # только один хост
```

---

## 3. Inventory

Из `[[DevOps/07_Ansible/inventory]]`:

```ini
[web]
web1 ansible_host=10.0.0.10 ansible_user=ubuntu
web2 ansible_host=10.0.0.11

[db]
db1 ansible_host=10.0.1.10

[prod:children]
web
db
```

YAML-формат и group_vars/host_vars — для переменных по группам и хостам.

---

## 4. Playbook и tasks

```yaml
- name: Configure web servers
  hosts: web
  become: true

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Deploy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

---

## 5. Roles

Из `[[DevOps/07_Ansible/playbooks-and-roles]]`:

```text
roles/
  common/
    tasks/main.yml
    handlers/main.yml
    templates/
    vars/main.yml
    defaults/main.yml
  web/
    tasks/main.yml
    defaults/main.yml
```

```yaml
- name: Common setup
  hosts: all
  roles:
    - common

- name: Web servers
  hosts: web
  roles:
    - web
```

---

## 6. Jinja2 и переменные

Из `[[DevOps/07_Ansible/jinja2-and-vars]]`:

```jinja2
server_name {{ server_name | default('_') }};
{% if ssl_enabled %}
ssl_certificate {{ ssl_cert_path }};
{% endif %}
{% for host in groups['web'] %}
server {{ host }}:{{ web_port }};
{% endfor %}
```

Фильтры: `default`, `trim`, `join`, `mandatory`.

---

## 7. Ansible Vault

Для секретов:

```bash
ansible-vault create group_vars/prod/vault.yml
ansible-playbook site.yml --ask-vault-pass
```

В vault.yml: `vault_db_password: "secret"`. В task: `{{ vault_db_password }}`.

См. `[[DevOps/10_Security/secrets-management]]`.

---

## 8. Dynamic inventory

Для облака (AWS, GCP, Azure) — плагины `amazon.aws.aws_ec2`, `google.cloud.gcp_compute` и т.д. Генерируют inventory из API облака. См. `[[DevOps/07_Ansible/inventory]]`.

---

## Что Middle DevOps должен знать по Ansible (чек‑лист)

- [ ] Умею писать inventory (INI/YAML), group_vars, host_vars (`[[DevOps/07_Ansible/inventory]]`).
- [ ] Пишу playbooks с tasks и handlers (`[[DevOps/07_Ansible/ansible-cheatsheet]]`).
- [ ] Создаю и использую roles (`[[DevOps/07_Ansible/playbooks-and-roles]]`).
- [ ] Применяю Jinja2 в templates (`[[DevOps/07_Ansible/jinja2-and-vars]]`).
- [ ] Использую Ansible Vault для секретов.
