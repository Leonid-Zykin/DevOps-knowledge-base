## Module 10 — Ansible Quiz

Ориентируйтесь на `[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`, `[[DevOps/07_Ansible/inventory]]`, `[[DevOps/07_Ansible/jinja2-and-vars]]`.

---

### Вопрос 1 — Idempotency

Что означает идемпотентность в Ansible?

a) Playbook выполняется только один раз  
b) Повторный запуск не меняет уже настроенное состояние; changed=0 при отсутствии изменений  
c) Ansible удаляет предыдущую конфигурацию  
d) Идемпотентность не применима к Ansible

---

### Вопрос 2 — Inventory

Где задаётся список хостов для Ansible?

a) В playbook  
b) В inventory (ini или yaml)  
c) В roles  
d) В ansible.cfg

---

### Вопрос 3 — Handler

Когда выполняется handler?

a) В начале playbook  
b) В конце каждого play  
c) Когда task с `notify` помечен как changed и все tasks play выполнены  
d) Handlers не выполняются автоматически

---

### Вопрос 4 — Role

Как подключить роль в playbook?

a) `include_role: web`  
b) `roles: [web]` в play  
c) `import: roles/web`  
d) Роли подключаются только через command line

---

### Вопрос 5 — group_vars

Где хранятся переменные для группы хостов?

a) В inventory.ini  
b) В `group_vars/<group_name>.yml`  
c) Только в playbook  
d) В ansible.cfg

---

### Вопрос 6 — --check

Что делает флаг `--check`?

a) Проверяет синтаксис playbook  
b) Режим dry-run: показывает изменения без применения  
c) Запускает только первый task  
d) Откатывает последний запуск

---

### Вопрос 7 — Ansible Vault

Для чего используется Ansible Vault?

a) Для шифрования всего playbook  
b) Для шифрования секретов (пароли, ключи) в переменных  
c) Для сжатия inventory  
d) Для кэширования фактов

---

### Вопрос 8 — become

Что делает директива `become: true`?

a) Повышает приоритет процесса  
b) Выполняет задачи от имени root (sudo)  
c) Запускает playbook в фоне  
d) become не существует

---

### Вопрос 9 — template модуль

Чем модуль `template` отличается от `copy`?

a) template быстрее  
b) template обрабатывает файл через Jinja2 перед копированием  
c) copy не поддерживает переменные  
d) Ничем

---

### Вопрос 10 — Dynamic inventory

Что такое dynamic inventory?

a) Inventory, который меняется во время выполнения  
b) Inventory, генерируемый из внешнего источника (API облака, скрипт)  
c) Inventory с переменными  
d) Inventory в формате YAML

---

> [!note]- Answers
> **1** — b) Повторный запуск не меняет уже настроенное; changed=0.  
> **2** — b) В inventory (ini или yaml).  
> **3** — c) Когда task с notify changed, в конце play.  
> **4** — b) `roles: [web]` в play.  
> **5** — b) В `group_vars/<group_name>.yml`.  
> **6** — b) Dry-run (режим проверки).  
> **7** — b) Шифрование секретов в переменных.  
> **8** — b) Выполнение от root (sudo).  
> **9** — b) template обрабатывает Jinja2.  
> **10** — b) Генерируется из внешнего источника (API облака).
