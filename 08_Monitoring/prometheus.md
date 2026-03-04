tags: [devops, monitoring-prometheus]

## Prometheus — PromQL, метрики, scrape, alerts

> ⚡ Tip: Думайте терминами метрик: **counter** (счётчик), **gauge** (измерение), **histogram/summary** (распределения/латентность).

### Типы метрик

| Тип       | Описание                                  | Примеры                          |
|----------|-------------------------------------------|----------------------------------|
| counter  | монотонно растёт                          | `http_requests_total`           |
| gauge    | может расти и падать                      | `memory_usage_bytes`            |
| histogram| bucket'ы + sum + count                    | `http_request_duration_seconds` |
| summary  | quantile'ы (на клиенте)                   | `rpc_duration_seconds`          |

### Базовые запросы PromQL

```promql
up                                           # 1, если target жив
rate(http_requests_total[5m])               # rps за 5 минут
sum(rate(http_requests_total[5m]))          # общий rps
sum by (method)(rate(http_requests_total[5m]))

node_memory_MemAvailable_bytes
  / node_memory_MemTotal_bytes

histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

Фильтры по лейблам:

```promql
http_requests_total{job="api", status=~"5.."}
rate(container_cpu_usage_seconds_total{namespace="prod"}[5m])
```

Арифметика:

```promql
sum(rate(http_requests_total[5m])) / sum(up)   # средний rps на инстанс
```

### Конфигурация Prometheus (scrape_configs)

`prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets:
          - 'node1:9100'
          - 'node2:9100'

  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
```

### Alerting rules

`alerts.yml`:

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
          description: "Target {{ $labels.job }} on {{ $labels.instance }} is down."

      - alert: HighCPU
        expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) > 0.9
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}"
```

Прокинуть правила в конфиг:

```yaml
rule_files:
  - "alerts.yml"
```

### Запуск Prometheus (docker-compose)

```yaml
version: "3.9"
services:
  prometheus:
    image: prom/prometheus:v2.53.1
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./alerts.yml:/etc/prometheus/alerts.yml:ro
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
    ports:
      - "9090:9090"
```

Связанные: `[[grafana]]`, `[[alertmanager]]`, `[[loki-and-logging]]`, `[[elk-stack]]`, `[[11_Networking/troubleshooting]]`.

## Gotchas

- **Использование `irate`/`rate` на слишком коротких/редких рядах**  
  - **Проблема**: шумные или нулевые значения, ложные алерты.  
  - **Решение**: выбирать окно `[...]` минимум в 4–5 scrape интервалов.

- **Слишком тяжёлые запросы в дашбордах/алертах**  
  - **Проблема**: большие `sum by`/`join` по миллионам серий убивают Prometheus.  
  - **Решение**: ограничивать кардинальность лейблов, использовать recording rules.

- **Лишние лейблы с высокой кардинальностью**  
  - **Проблема**: каждый уникальный набор лейблов = новая временная серия → взрыв данных.  
  - **Решение**: не добавлять в метки динамичные значения (ID запросов, email, UUID).

> ⚠️ Warning: Prometheus не БД общего назначения. Хранение логов/сырых trace'ов или данных с крайне высокой кардинальностью (user_id, request_id) может привести к неконтролируемому росту объёма и деградации производительности. Для логов используйте Loki/ELK (`[[loki-and-logging]]`, `[[elk-stack]]`).

