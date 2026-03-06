## Module 13 — Prometheus & Grafana

### Почему это важно

Без метрик и алертов вы не видите состояние системы до того, как пользователи начнут жаловаться. Prometheus — стандарт для метрик в Kubernetes; Grafana — визуализация и алерты. Middle DevOps должен уметь настраивать мониторинг и реагировать на алерты.

Опираемся на:
- `[[DevOps/08_Monitoring/prometheus]]`
- `[[DevOps/08_Monitoring/grafana]]`
- `[[DevOps/08_Monitoring/alertmanager]]`

---

## 1. Типы метрик Prometheus

Из `[[DevOps/08_Monitoring/prometheus]]`:

| Тип       | Описание                    | Примеры                    |
|-----------|-----------------------------|----------------------------|
| counter   | монотонно растёт            | `http_requests_total`      |
| gauge     | может расти и падать        | `memory_usage_bytes`       |
| histogram | bucket'ы + sum + count      | `http_request_duration_seconds` |
| summary   | quantile'ы (на клиенте)     | `rpc_duration_seconds`    |

---

## 2. Базовые PromQL-запросы

```promql
up
rate(http_requests_total[5m])
sum(rate(http_requests_total[5m])) by (method)
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

Фильтры: `{job="api", status=~"5.."}`.

---

## 3. Scrape config

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['node1:9100', 'node2:9100']
```

Для Kubernetes — `kubernetes_sd_configs` с relabel_configs.

---

## 4. Alerting rules

```yaml
groups:
  - name: instance
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
```

Alertmanager — маршрутизация, группировка, уведомления (Slack, PagerDuty, email).

---

## 5. Grafana

Из `[[DevOps/08_Monitoring/grafana]]`:

- **Datasource** — Prometheus, Loki и др.
- **Dashboard** — панели с запросами PromQL.
- **Variables** — `label_values(up, job)` для фильтров.
- **Alerting** — правила на основе запросов, contact points.

---

## Что Middle DevOps должен знать (чек‑лист)

- [ ] Понимаю типы метрик и базовый PromQL (`[[DevOps/08_Monitoring/prometheus]]`).
- [ ] Умею настраивать scrape_configs.
- [ ] Строю дашборды в Grafana (`[[DevOps/08_Monitoring/grafana]]`).
- [ ] Настраиваю алерты (Prometheus rules + Alertmanager или Grafana).
