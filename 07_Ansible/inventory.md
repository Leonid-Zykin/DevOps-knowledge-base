tags: [devops, ansible-inventory]

## Ansible Inventory — static, dynamic, groups, vars

> ⚡ Tip: Инвентори описывает, какие хосты есть и как к ним подключаться. Используйте группы и vars для DRY.

### Статический INI inventory

`inventory.ini`:

```ini
[web]
web1 ansible_host=10.0.0.10 ansible_user=ubuntu
web2 ansible_host=10.0.0.11 ansible_user=ubuntu

[db]
db1 ansible_host=10.0.1.10 ansible_user=postgres

[prod:children]
web
db

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Проверка:

```bash
ansible web -i inventory.ini -m ping
ansible prod -i inventory.ini -m shell -a "hostname"
```

### YAML inventory

`inventory.yml`:

```yaml
all:
  vars:
    ansible_user: ubuntu
  children:
    web:
      hosts:
        web1:
          ansible_host: 10.0.0.10
        web2:
          ansible_host: 10.0.0.11
    db:
      hosts:
        db1:
          ansible_host: 10.0.1.10
          ansible_user: postgres
```

Запуск:

```bash
ansible web -i inventory.yml -m ping
```

### group_vars и host_vars

```text
inventory.ini
group_vars/
  all.yml
  web.yml
  db.yml
host_vars/
  web1.yml
```

`group_vars/all.yml`:

```yaml
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org
```

`group_vars/web.yml`:

```yaml
web_port: 80
```

`host_vars/web1.yml`:

```yaml
web_port: 8080
```

### Dynamic inventory (пример AWS EC2)

Установка плагина (для старых версий Ansible):

```bash
pip install boto3 botocore
```

`aws_ec2.yml`:

```yaml
plugin: aws_ec2
regions:
  - eu-central-1
filters:
  tag:Environment: prod
keyed_groups:
  - key: tags.Role
    prefix: role_
hostnames:
  - tag:Name
compose:
  ansible_host: public_ip_address
```

Запуск:

```bash
ansible-inventory -i aws_ec2.yml --graph
ansible all -i aws_ec2.yml -m ping
```

Связанные: `[[ansible-cheatsheet]]`, `[[playbooks-and-roles]]`, `[[jinja2-and-vars]]`.

## Gotchas

- **Смешивание параметров подключения в разных местах**  
  - **Проблема**: трудно понять, откуда берётся `ansible_user`/`ansible_host`.  
  - **Решение**: договориться, где хранить подключенческие переменные (в инвентори vs group_vars/host_vars) и документировать.

- **Случайное применение playbook ко всем хостам**  
  - **Проблема**: `hosts: all` в playbook и `-i aws_ec2.yml` без фильтров — действия по всем prod‑нодам.  
  - **Решение**: использовать более точные группы (`prod_web`, `prod_db`) и `--limit` при запуске.

- **Dynamic inventory без чётких фильтров**  
  - **Проблема**: в группу попадают лишние хосты (dev/stage).  
  - **Решение**: фильтровать по тегам/меткам (`Environment=prod`, `Role=web`) и регулярно делать `ansible-inventory --graph` для проверки.

> ⚠️ Warning: Любые изменения в inventory для прод‑хостов (особенно в dynamic inventory) могут затронуть группы, на которые ориентируются критичные playbook'и (обновления, рестарты сервисов). Перед применением сложных playbook'ов всегда проверяйте, какие хосты попадут под `hosts`/`--limit`.

