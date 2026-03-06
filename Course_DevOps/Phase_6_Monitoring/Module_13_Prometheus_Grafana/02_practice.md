## Module 13 — Prometheus & Grafana Practice

Нужен Prometheus, Grafana и хотя бы один экспортер (node_exporter, или приложение с /metrics). Можно использовать Docker Compose или Kubernetes.

Опирайтесь на `[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`.

---

### Task 1 — Запуск Prometheus

- **Goal**: Запустить Prometheus и открыть UI.
- **Steps**:
  1. Создайте `prometheus.yml` с job `prometheus` (localhost:9090).
  2. Запустите Prometheus (Docker или бинарь).
  3. Откройте http://localhost:9090, проверьте Status → Targets.
- **Expected result**: Prometheus работает, target up.

---

### Task 2 — Node exporter

- **Goal**: Собирать метрики хоста.
- **Steps**:
  1. Запустите node_exporter на порту 9100.
  2. Добавьте job в prometheus.yml.
  3. Перезапустите Prometheus, проверьте метрики `node_*`.
- **Expected result**: Метрики CPU, memory, disk доступны.

---

### Task 3 — PromQL: rate и sum

- **Goal**: Написать запросы.
- **Steps**:
  1. Выполните `rate(node_cpu_seconds_total[5m])`
  2. Выполните `sum(rate(node_cpu_seconds_total{mode!="idle"}[5m]))`
  3. Выполните `node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes`
- **Expected result**: Понимание rate, sum, арифметики.

---

### Task 4 — Scrape с лейблами

- **Goal**: Добавить статические лейблы.
- **Steps**:
  1. В static_configs добавьте `labels: { env: "dev" }`
  2. Проверьте, что метрики имеют лейбл env.
- **Expected result**: Кастомные лейблы в метриках.

---

### Task 5 — Alerting rule

- **Goal**: Создать правило алерта.
- **Steps**:
  1. Создайте `alerts.yml` с правилом (например, InstanceDown или HighCPU).
  2. Подключите в prometheus.yml: `rule_files: [alerts.yml]`
  3. Проверьте Alerts в UI.
- **Expected result**: Алерт появляется при срабатывании условия.

---

### Task 6 — Grafana и Prometheus datasource

- **Goal**: Подключить Grafana к Prometheus.
- **Steps**:
  1. Запустите Grafana.
  2. Добавьте datasource Prometheus (URL: http://prometheus:9090).
  3. Выполните тестовый запрос.
- **Expected result**: Grafana видит Prometheus.

---

### Task 7 — Простой дашборд

- **Goal**: Создать панель с графиком.
- **Steps**:
  1. Создайте Dashboard, добавьте Panel.
  2. Query: `rate(node_cpu_seconds_total{mode!="idle"}[5m])`
  3. Сохраните дашборд.
- **Expected result**: График CPU в реальном времени.

---

### Task 8 — Импорт дашборда

- **Goal**: Использовать готовый дашборд.
- **Steps**:
  1. Зайдите в Grafana → Dashboards → Import.
  2. Введите ID 1860 (Node Exporter Full) или 3662 (Kubernetes cluster).
  3. Выберите Prometheus datasource.
- **Expected result**: Готовый дашборд с множеством панелей.

---

### Task 9 — Variables

- **Goal**: Добавить переменную для фильтрации.
- **Steps**:
  1. Создайте variable типа Query: `label_values(up, instance)`
  2. В панели используйте `{instance=~"$instance"}`
  3. Проверьте dropdown в дашборде.
- **Expected result**: Динамический фильтр по instance.

---

### Task 10 — Break it: тяжёлый запрос

- **Goal**: Понять влияние тяжёлых запросов.
- **Steps**:
  1. Создайте запрос с большим range: `rate(metric[24h])` на высококардинальной метрике.
  2. Наблюдайте нагрузку на Prometheus.
  3. Оптимизируйте: уменьшите range, используйте recording rules.
- **Expected result**: Понимание, что тяжёлые запросы могут нагружать Prometheus.

---

## Итог

После практики вы умеете:
- настраивать Prometheus и scrape_configs;
- писать PromQL-запросы;
- строить дашборды в Grafana;
- создавать алерты.
