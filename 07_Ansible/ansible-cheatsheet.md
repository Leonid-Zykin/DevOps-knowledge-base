tags: [devops, ansible]

## Ansible Cheatsheet — ad-hoc, playbook, vault

> ⚡ Tip: Ansible хорошо подходит для идемпотентной конфигурации серверов и простых оркестраций. Для сложной инфраструктуры см. `[[06_Terraform/terraform-cheatsheet]]`.

### Базовые команды

```bash
ansible --version
ansible all -m ping -i inventory.ini
ansible-playbook site.yml -i inventory.ini

ansible-config dump | less
ansible-doc -l                         # список модулей
ansible-doc yum                        # справка по модулю
```

### Инвентори (ini и YAML)

`inventory.ini`:

```ini
[web]
web1 ansible_host=10.0.0.10
web2 ansible_host=10.0.0.11

[db]
db1 ansible_host=10.0.1.10

[prod:children]
web
db
```

`inventory.yml`:

```yaml
all:
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
```

Подробнее: `[[inventory]]`.

### Ad-hoc команды

```bash
ansible all -i inventory.ini -m ping

ansible web -i inventory.ini -m shell -a "uptime"
ansible web -b -m apt -a "name=nginx state=present update_cache=yes"

ansible db -b -m service -a "name=postgresql state=restarted"
```

Флаги:

- `-i` — инвентори;
- `-u` — пользователь SSH;
- `-b` — become (sudo);
- `-k` — спросить пароль SSH;
- `-K` — спросить пароль sudo.

### Простой playbook

`site.yml`:

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

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

Запуск:

```bash
ansible-playbook -i inventory.ini site.yml
```

### Tags и выбор задач

```yaml
tasks:
  - name: Install packages
    apt:
      name: nginx
      state: present
    tags: [packages]

  - name: Deploy config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx
    tags: [config]

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

```bash
ansible-playbook site.yml --tags packages
ansible-playbook site.yml --skip-tags config
```

### Факты (facts)

```bash
ansible all -m setup -i inventory.ini           # все факты
ansible web1 -m setup -a 'filter=ansible_os_*'
```

В playbook:

```yaml
- debug:
    msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

### Vault (шифрование секретов)

Создать зашифрованный файл:

```bash
ansible-vault create group_vars/prod/vault.yml
ansible-vault edit group_vars/prod/vault.yml
ansible-vault view group_vars/prod/vault.yml
ansible-vault rekey group_vars/prod/vault.yml
```

Использование:

```bash
ansible-playbook site.yml --ask-vault-pass
# или
ansible-playbook site.yml --vault-password-file ~/.ansible_vault_pass
```

`group_vars/prod/vault.yml`:

```yaml
vault_db_password: "secret"
```

В task:

```yaml
- name: Template config
  template:
    src: app.env.j2
    dest: /etc/app/env
  vars:
    db_password: "{{ vault_db_password }}"
```

Связанные заметки: `[[playbooks-and-roles]]`, `[[jinja2-and-vars]]`, `[[inventory]]`, `[[10_Security/secrets-management]]`.

## Gotchas

- **Запуск destructiv‑тасков без dry‑run/лимита хостов**  
  - **Проблема**: неверная команда (`rm -rf`, массовый `service stop`) применяется ко всем хостам группы.  
  - **Решение**: сначала ограничить `--limit host1`, использовать `--check` для idempotent изменений, тщательно ревьюить таски.

- **Секреты без Vault**  
  - **Проблема**: пароли и ключи лежат в чистом YAML в Git.  
  - **Решение**: использовать Ansible Vault или внешние секрет‑хранилища (`[[10_Security/secrets-management]]`).

- **Игнорирование idempotency**  
  - **Проблема**: таски всегда помечаются как changed, переустанавливают пакеты, перезапускают сервисы.  
  - **Решение**: использовать корректные модули (apt/yum/service/file/template) вместо `shell/command`, проверять `changed_when`.

> ⚠️ Warning: `shell`/`command` модули Ansible без чёткого `creates`/`removes`/`changed_when` превращают playbook в набор непредсказуемых скриптов. На проде это может привести к неконтролируемым перезапускам сервисов и порче конфигурации — по возможности используйте специализированные модули.

