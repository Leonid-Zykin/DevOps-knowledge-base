## Module 10 — Ansible Practice

Ansible установлен. Нужны минимум 1–2 целевых хоста (VM, Docker-контейнеры с SSH или localhost). Для Docker можно использовать образ с SSH или `ansible_connection=local` для localhost.

Опирайтесь на `[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`, `[[DevOps/07_Ansible/inventory]]`, `[[DevOps/07_Ansible/jinja2-and-vars]]`.

---

### Task 1 — Ping и ad-hoc

- **Goal**: Проверить связь с хостами.
- **Steps**:
  1. Создайте `inventory.ini` с одной или несколькими группами (можно localhost с `ansible_connection=local`).
  2. Выполните: `ansible all -i inventory.ini -m ping`
  3. Выполните ad-hoc: `ansible all -i inventory.ini -m shell -a "hostname"`
- **Expected result**: Все хосты отвечают, hostname выводится.

---

### Task 2 — Простой playbook

- **Goal**: Установить пакет и запустить сервис.
- **Steps**:
  1. Создайте `site.yml` с play: hosts, become, tasks (apt install, service started).
  2. Установите, например, nginx или htop.
  3. Запустите: `ansible-playbook -i inventory.ini site.yml`
  4. Повторите запуск — убедитесь, что changed=0 (идемпотентность).
- **Expected result**: Пакет установлен, сервис запущен; повторный запуск не меняет состояние.

---

### Task 3 — Template и handlers

- **Goal**: Развернуть конфиг через шаблон.
- **Steps**:
  1. Создайте `templates/nginx.conf.j2` (или аналог) с переменными.
  2. В playbook добавьте task `template` с `notify: restart nginx`.
  3. Добавьте handler `restart nginx`.
  4. Запустите playbook, измените шаблон, запустите снова — handler должен сработать.
- **Expected result**: Конфиг деплоится, при изменении сервис перезапускается.

---

### Task 4 — group_vars и host_vars

- **Goal**: Параметризовать через переменные.
- **Steps**:
  1. Создайте `group_vars/all.yml` и `group_vars/web.yml` с переменными.
  2. Используйте их в шаблоне (например, `web_port`, `server_name`).
  3. Добавьте `host_vars/web1.yml` с переопределением для одного хоста.
  4. Проверьте через `ansible-inventory -i inventory.ini --list -y`
- **Expected result**: Разные значения для групп и хостов.

---

### Task 5 — Role

- **Goal**: Вынести логику в роль.
- **Steps**:
  1. Создайте структуру: `roles/web/tasks/main.yml`, `roles/web/defaults/main.yml`, `roles/web/templates/`.
  2. Перенесите tasks и template в роль.
  3. В playbook замените tasks на `roles: [web]`
  4. Запустите playbook.
- **Expected result**: Роль применяется, playbook остаётся тонким.

---

### Task 6 — Условная логика в шаблоне

- **Goal**: Использовать if/else в Jinja2.
- **Steps**:
  1. В шаблоне добавьте `{% if ssl_enabled %}...{% endif %}`
  2. В group_vars задайте `ssl_enabled: false` и `ssl_enabled: true` для разных групп.
  3. Запустите для обеих групп и проверьте сгенерированный конфиг.
- **Expected result**: Конфиг меняется в зависимости от переменной.

---

### Task 7 — Tags

- **Goal**: Запускать только часть задач.
- **Steps**:
  1. Добавьте tags к tasks: `tags: [packages]`, `tags: [config]`
  2. Запустите: `ansible-playbook site.yml --tags packages`
  3. Запустите: `ansible-playbook site.yml --skip-tags config`
- **Expected result**: Выполняются только задачи с выбранными тегами.

---

### Task 8 — Ansible Vault

- **Goal**: Зашифровать секреты.
- **Steps**:
  1. Создайте `ansible-vault create group_vars/prod/vault.yml` с паролем БД.
  2. В task используйте `{{ vault_db_password }}`
  3. Запустите playbook с `--ask-vault-pass`
  4. Проверьте, что секрет не виден в `ansible-inventory` без пароля.
- **Expected result**: Секреты зашифрованы, playbook работает с vault.

---

### Task 9 — --check и --diff

- **Goal**: Dry-run перед применением.
- **Steps**:
  1. Запустите `ansible-playbook site.yml --check --diff`
  2. Измените шаблон и снова запустите с --check --diff
  3. Убедитесь, что видите diff без реальных изменений на хостах.
- **Expected result**: Понимание, как проверять изменения до apply.

---

### Task 10 — Break it: неидемпотентный task

- **Goal**: Понять важность идемпотентности.
- **Steps**:
  1. Добавьте task с `command: echo "test" >> /tmp/log.txt` (без creates/removes).
  2. Запустите playbook несколько раз — каждый раз будет changed.
  3. Замените на идемпотентный вариант (например, lineinfile или copy с определённым содержимым).
  4. Повторите — changed только при первом запуске.
- **Expected result**: Понимание, почему нужны идемпотентные модули.

---

## Итог

После практики вы умеете:
- писать inventory и playbooks;
- использовать roles, templates, handlers;
- работать с переменными и Vault;
- проверять изменения через --check --diff.
