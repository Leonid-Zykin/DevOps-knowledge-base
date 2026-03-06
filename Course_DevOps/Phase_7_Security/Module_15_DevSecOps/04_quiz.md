## Module 15 — DevSecOps Quiz

Ориентируйтесь на `[[DevOps/10_Security/devsecops]]`, `[[DevOps/10_Security/secrets-management]]`, `[[DevOps/10_Security/container-security]]`.

---

### Вопрос 1 — SAST

Что такое SAST?

a) Сканирование образов  
b) Статический анализ кода (без запуска приложения)  
c) Динамическое тестирование  
d) Сканирование зависимостей

---

### Вопрос 2 — Trivy

Для чего используется Trivy?

a) Только для логов  
b) Сканирование образов, файловой системы, конфигов на уязвимости  
c) Только для Terraform  
d) Trivy не используется в DevSecOps

---

### Вопрос 3 — SOPS

Для чего используется SOPS?

a) Для сканирования кода  
b) Для шифрования YAML/JSON файлов с секретами (PGP, KMS)  
c) Для мониторинга  
d) SOPS не используется

---

### Вопрос 4 — SealedSecrets

Что такое SealedSecrets в Kubernetes?

a) Секреты с истекающим сроком  
b) Зашифрованная версия Secret, которую можно хранить в Git; controller расшифровывает в кластере  
c) Секреты только для одного namespace  
d) SealedSecrets не существует

---

### Вопрос 5 — runAsNonRoot

Что делает `runAsNonRoot: true` в securityContext?

a) Запрещает root в контейнере  
b) Разрешает только root  
c) Отключает capabilities  
d) runAsNonRoot не существует

---

### Вопрос 6 — readOnlyRootFilesystem

Зачем нужен readOnlyRootFilesystem?

a) Чтобы ускорить загрузку  
b) Чтобы предотвратить запись в корневую ФС контейнера (защита от модификации)  
c) Чтобы сжать образ  
d) readOnlyRootFilesystem не используется

---

### Вопрос 7 — Security gate

Что такое security gate в CI?

a) Отдельный сервер для безопасности  
b) Проверка, блокирующая merge/deploy при критичных уязвимостях  
c) Firewall в облаке  
d) Security gate не существует

---

### Вопрос 8 — allowPrivilegeEscalation

Что делает `allowPrivilegeEscalation: false`?

a) Запрещает процессу повышать привилегии (например, через setuid)  
b) Разрешает root  
c) Отключает сеть  
d) allowPrivilegeEscalation не существует

---

### Вопрос 9 — capabilities drop

Что делает `capabilities: drop: ["ALL"]`?

a) Добавляет все capabilities  
b) Удаляет все Linux capabilities у контейнера  
c) Удаляет только NET_ADMIN  
d) capabilities не поддерживаются в K8s

---

### Вопрос 10 — Best practice для секретов

Где НЕ должны храниться секреты?

a) В Vault  
b) В Git в открытом виде, в логах, в Docker-образах  
c) В Kubernetes Secret  
d) Секреты можно хранить где угодно

---

> [!note]- Answers
> **1** — b) Статический анализ кода.  
> **2** — b) Сканирование образов, FS, конфигов.  
> **3** — b) Шифрование файлов с секретами.  
> **4** — b) Зашифрованный Secret для Git, controller расшифровывает.  
> **5** — a) Запрещает root.  
> **6** — b) Предотвращение записи в корневую ФС.  
> **7** — b) Блокировка при уязвимостях.  
> **8** — a) Запрещает повышение привилегий.  
> **9** — b) Удаляет все capabilities.  
> **10** — b) Не в Git открыто, не в логах, не в образах.
