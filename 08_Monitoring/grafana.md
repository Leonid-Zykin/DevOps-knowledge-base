tags: [devops, grafana]

## Grafana — дашборды, datasource, alerting

> ⚡ Tip: Grafana = визуальный frontend для Prometheus/Loki/Influx и др. Начинайте с импортов готовых дашбордов, затем кастомизируйте.

### Настройка datasources

В UI: Configuration → Data sources → Add.

Пример `datasource.yaml` (provisioning):

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
```

### Запуск Grafana (docker-compose)

```yaml
version: "3.9"
services:
  grafana:
    image: grafana/grafana:11.0.0
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./provisioning:/etc/grafana/provisioning
```

### Построение графиков (Prometheus datasource)

- Query: `rate(http_requests_total[5m])`;
- Legend: `{{method}} {{status}}`;
- Panel type: Time series.

Шаблон (variables):

- Type: Query;
- Data source: Prometheus;
- Query: `label_values(up, job)`;
- Name: `job`.

Использование в запросах:

```promql
rate(http_requests_total{job="$job"}[5m])
```

### Alerting (Grafana 8+ unified alerting)

1. Alert rules → New alert rule.
2. Query (Prometheus), условие (threshold).
3. Notification policies + contact points (Slack/Email/Webhook).

Пример Prometheus‑запроса:

```promql
avg_over_time(rate(http_requests_total{job="api",status=~"5.."}[5m])[15m:]) > 1
```

Связанные: `[[prometheus]]`, `[[alertmanager]]`, `[[loki-and-logging]]`.

## Gotchas

- **Дешборды с тяжёлыми запросами**  
  - **Проблема**: один «тяжёлый» дашборд может уронить Prometheus.  
  - **Решение**: использовать recording rules, ограничивать период запросов и количество серий.

- **Разрозненные дашборды без стандартов**  
  - **Проблема**: каждый сервис имеет свой уникальный дашборд, сложно ориентироваться.  
  - **Решение**: создать базовый шаблон дашборда для сервисов (SLI: latency, errors, traffic, saturation) и наследовать его.

- **Alerting только в Grafana без Alertmanager**  
  - **Проблема**: сложнее управлять маршрутизацией и подавлением алертов.  
  - **Решение**: для Prometheus‑алармов использовать Alertmanager (`[[alertmanager]]`), Grafana alerts — для дополнительных случаев.

> ⚠️ Warning: Избыточное количество алертов (alert fatigue) приводит к игнорированию уведомлений и пропуску реальных инцидентов. Перед вводом нового алерта определяйте SLO/порог, канал уведомления и процедуру реакции (`[[12_Soft-Skills-and-Processes/incident-response]]`, `[[12_Soft-Skills-and-Processes/sre-concepts]]`).

