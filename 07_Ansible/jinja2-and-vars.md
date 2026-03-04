tags: [devops, ansible-jinja2]

## Ansible Jinja2 и переменные — шаблоны, фильтры, условия

> ⚡ Tip: Jinja2 используется во всех шаблонах (`template`) и при построении строк/выражений в Ansible. Проверяйте шаблоны через `ansible-playbook --check` и `--diff`.

### Переменные в шаблонах

```jinja2
APP_ENV={{ env }}
PORT={{ web_port }}
```

Playbook:

```yaml
- name: Render env file
  template:
    src: app.env.j2
    dest: /etc/app/app.env
  vars:
    env: prod
```

Источники переменных:

- group_vars/host_vars;
- vars в play/role;
- facts (`ansible_` переменные).

### Условия в шаблонах

```jinja2
{% if env == "prod" %}
log_level=info
{% else %}
log_level=debug
{% endif %}
```

### Циклы

```jinja2
{% for host in groups['web'] %}
server {{ host }}:{{ web_port }};
{% endfor %}
```

### Частые фильтры

```jinja2
{{ my_list | join(',') }}
{{ 'Prod' | lower }}
{{ '  value  ' | trim }}
{{ 123 | string }}

{{ var | default('fallback') }}
{{ var | mandatory }}   {# ошибка, если var не определён #}
```

Работа со словарями:

```jinja2
{% for k, v in my_dict.items() %}
{{ k }}={{ v }}
{% endfor %}
```

### Генерация конфигов

`nginx.conf.j2`:

```jinja2
server {
  listen {{ web_port }};
  server_name {{ server_name | default('_') }};

  {% if ssl_enabled %}
  ssl_certificate     {{ ssl_cert_path }};
  ssl_certificate_key {{ ssl_key_path }};
  {% endif %}
}
```

Связанные: `[[ansible-cheatsheet]]`, `[[playbooks-and-roles]]`, `[[inventory]]`.

## Gotchas

- **Ошибки шаблонов на проде**  
  - **Проблема**: некорректный Jinja2 ломает конфиг (nginx/postgres), сервис не стартует.  
  - **Решение**: использовать `--check --diff`, генерировать файл в tmp и валидировать (`nginx -t`, `postgres -t`) перед заменой боевого.

- **Неинициализированные переменные**  
  - **Проблема**: опечатка в имени переменной приводит к пустому значению.  
  - **Решение**: для критичных переменных использовать `| mandatory`, для остальных — `| default(...)`.

- **Смешивание логики и представления**  
  - **Проблема**: сложная бизнес‑логика в шаблонах делает их нечитаемыми.  
  - **Решение**: готовить данные на стороне Ansible (set_fact, vars), а в шаблоне только выводить.

> ⚠️ Warning: Шаблоны конфигураций для критичных сервисов (nginx, HAProxy, PostgreSQL и т.п.) должны проходить автоматизированные проверки (lint, `*-t` validate) в CI и на staging, прежде чем попадать на прод. Ошибка в одном шаблоне может положить весь входящий трафик.

