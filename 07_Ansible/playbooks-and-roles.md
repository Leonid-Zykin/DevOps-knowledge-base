tags: [devops, ansible-playbooks]

## Ansible Playbooks и Roles — структура и примеры

> ⚡ Tip: Держите логику в ролях, а playbook'и делайте тонкими, описывая только «что где применять».

### Структура проекта с ролями

```text
inventory.ini
site.yml
roles/
  common/
    tasks/main.yml
    handlers/main.yml
    templates/
    files/
    vars/main.yml
    defaults/main.yml
    meta/main.yml
  web/
    ...
  db/
    ...
group_vars/
  all.yml
  prod.yml
  dev.yml
host_vars/
  web1.yml
```

### Пример `site.yml`

```yaml
- name: Common configuration
  hosts: all
  become: true
  roles:
    - common

- name: Web servers
  hosts: web
  become: true
  roles:
    - web

- name: DB servers
  hosts: db
  become: true
  roles:
    - db
```

### Role `common`

`roles/common/tasks/main.yml`:

```yaml
- name: Ensure basic packages are installed
  apt:
    name:
      - htop
      - curl
      - git
    state: present
    update_cache: yes

- name: Set timezone
  timezone:
    name: Europe/Berlin
```

### Role `web`

`roles/web/defaults/main.yml`:

```yaml
web_port: 80
```

`roles/web/tasks/main.yml`:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

`roles/web/handlers/main.yml`:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

`roles/web/templates/nginx.conf.j2`:

```jinja2
server {
  listen {{ web_port }};
  server_name _;

  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
```

### group_vars и host_vars

`group_vars/prod.yml`:

```yaml
web_port: 8080
env: prod
```

`host_vars/web1.yml`:

```yaml
web_port: 8081
```

Связанные заметки: `[[ansible-cheatsheet]]`, `[[inventory]]`, `[[jinja2-and-vars]]`.

## Gotchas

- **Логика в playbook вместо roles**  
  - **Проблема**: playbook разрастается, сложно переиспользовать.  
  - **Решение**: выделять повторяемый функционал в роли, использовать их в разных playbook'ах/проектах.

- **Смешивание group_vars/host_vars и inline vars**  
  - **Проблема**: трудно понять, откуда берётся значение переменной.  
  - **Решение**: договориться о порядке приоритета и документировать, где задаются какие переменные.

- **Отсутствие defaults в ролях**  
  - **Проблема**: роль не самодостаточна, требует кучи внешних переменных.  
  - **Решение**: задавать разумные значения по умолчанию в `defaults/main.yml`, а переопределения — в group_vars/host_vars.

> ⚠️ Warning: Слишком «умные» универсальные роли с большим количеством ветвлений и скрытой магии усложняют сопровождение. Лучше больше маленьких и простых ролей с чётко задокументированными переменными, чем один монолитный «do-everything» role.

