## Module 14 — Logging Practice

Нужны Loki, Promtail и Grafana. Можно использовать Docker Compose или Helm (loki-stack, promtail).

Опирайтесь на `[[DevOps/08_Monitoring/loki-and-logging]]`.

---

### Task 1 — Запуск Loki

- **Goal**: Запустить Loki.
- **Steps**:
  1. Скачайте конфиг Loki (минимальный) или используйте готовый.
  2. Запустите Loki (Docker или бинарь).
  3. Проверьте http://localhost:3100/ready
- **Expected result**: Loki отвечает.

---

### Task 2 — Promtail и файлы

- **Goal**: Собирать логи из файлов.
- **Steps**:
  1. Создайте promtail-config.yaml с job для `/var/log/*.log` или тестового файла.
  2. Запустите Promtail.
  3. Запишите что-то в лог-файл.
  4. Проверьте в Grafana (Loki datasource), что логи появились.
- **Expected result**: Логи из файла видны в Loki.

---

### Task 3 — LogQL: фильтр по лейблам

- **Goal**: Запрашивать логи по лейблам.
- **Steps**:
  1. Выполните `{job="varlogs"}` в Grafana Explore.
  2. Добавьте фильтр `|= "ERROR"` или `|~ "error|fail"`.
  3. Проверьте, что результаты фильтруются.
- **Expected result**: Понимание синтаксиса LogQL.

---

### Task 4 — Парсинг JSON

- **Goal**: Парсить JSON-логи.
- **Steps**:
  1. Создайте лог-файл с JSON (например, `{"level":"error","msg":"fail"}`).
  2. Настройте Promtail для этого файла.
  3. Запрос: `{job="..."} | json | level="error"`
- **Expected result**: Фильтрация по полям JSON.

---

### Task 5 — Метрики из логов

- **Goal**: Считать rate ошибок.
- **Steps**:
  1. Запрос: `sum(rate({app="api"} |= "ERROR" [5m]))`
  2. Используйте в Grafana как метрику (или в Prometheus через Loki metrics).
- **Expected result**: Метрика «количество ошибок в секунду» из логов.

---

### Task 6 — Promtail в Kubernetes

- **Goal**: Собирать логи контейнеров.
- **Steps**:
  1. Установите Promtail как DaemonSet (Helm или манифесты).
  2. Настройте pipeline для чтения `/var/log/containers/*.log`.
  3. Добавьте лейблы из метаданных Pod (namespace, pod, container).
  4. Проверьте логи в Grafana по namespace/pod.
- **Expected result**: Логи всех подов в Loki.

---

### Task 7 — Grafana Explore

- **Goal**: Эффективно искать логи.
- **Steps**:
  1. Добавьте Loki как datasource.
  2. В Explore выберите Loki.
  3. Используйте Live tail для потокового просмотра.
  4. Сохраните запрос в дашборд.
- **Expected result**: Уверенный поиск в Grafana.

---

### Task 8 — Дашборд с логами

- **Goal**: Добавить панель с логами в дашборд.
- **Steps**:
  1. Создайте панель типа Logs.
  2. Datasource: Loki, Query: `{namespace="default"}`
  3. Добавьте переменную для выбора namespace.
- **Expected result**: Дашборд с логами.

---

### Task 9 — Break it: высокая кардинальность

- **Goal**: Понять ограничения лейблов.
- **Steps**:
  1. Не используйте высококардинальные значения (user_id, request_id) как лейблы.
  2. Используйте их в фильтрах после парсинга: `| json | user_id="123"`
  3. Обсудите, почему лейблы должны быть низкокардинальными.
- **Expected result**: Понимание best practices Loki.

---

### Task 10 — Интеграция с приложением

- **Goal**: Собирать логи приложения.
- **Steps**:
  1. Настройте приложение писать логи в stdout (или файл).
  2. Promtail собирает их (из контейнера или файла).
  3. Создайте запросы для типичных сценариев (ошибки, медленные запросы).
- **Expected result**: Полный цикл: приложение → Promtail → Loki → Grafana.

---

## Итог

После практики вы умеете:
- настраивать Loki и Promtail;
- писать LogQL-запросы;
- интегрировать с Kubernetes;
- использовать Grafana для логов.
