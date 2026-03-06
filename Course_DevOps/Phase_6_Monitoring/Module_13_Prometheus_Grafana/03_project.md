## Module 13 — Mini-project: Monitoring Stack

### 1. Цель проекта

**Задача**: развернуть стек мониторинга:

- Prometheus (сбор метрик);
- Grafana (дашборды);
- Alertmanager (опционально);
- Экспортеры для приложения и инфраструктуры.

Оценочное время: **4–8 часов**.

---

### 2. Требования

1. **Prometheus**:
   - Scrape node_exporter, приложения с /metrics (если есть).
   - Для K8s: kubernetes_sd_configs или ServiceMonitor.
2. **Grafana**:
   - Datasource Prometheus.
   - Минимум 2 дашборда: инфраструктура (CPU, memory, disk) и приложение (если есть метрики).
3. **Алерты**:
   - InstanceDown, HighCPU, HighMemory (или аналог).
   - Alertmanager с уведомлением (Slack/email или лог).
4. **Деплой**:
   - Docker Compose или Kubernetes (Helm charts: kube-prometheus-stack, prometheus-operator).

---

### 3. Пошаговый план

#### Шаг 1 — Инфраструктура

- Запустите Prometheus, node_exporter.
- Настройте scrape_configs.

#### Шаг 2 — Grafana

- Установите Grafana, добавьте Prometheus.
- Импортируйте Node Exporter Full (1860).

#### Шаг 3 — Метрики приложения

- Если есть приложение — добавьте /metrics (Prometheus client library).
- Добавьте job в Prometheus.
- Создайте дашборд для приложения.

#### Шаг 4 — Алерты

- Создайте правила: InstanceDown, HighCPU.
- Настройте Alertmanager (если нужны уведомления).

#### Шаг 5 — Документация

- README с описанием стека и как его запускать.
- Список алертов и что они означают.

---

### 4. Acceptance criteria

- [ ] Prometheus собирает метрики
- [ ] Grafana отображает дашборды
- [ ] Есть минимум 2 алерта
- [ ] Стек можно развернуть одной командой (docker-compose up или helm install)

---

### 5. Stretch goals

- Kubernetes: развернуть через Helm (kube-prometheus-stack).
- Recording rules для тяжёлых запросов.
- Уведомления в Slack/Telegram.

---

См. `[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`, `[[DevOps/08_Monitoring/alertmanager]]`.
