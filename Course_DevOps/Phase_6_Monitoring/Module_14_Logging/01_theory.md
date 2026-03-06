## Module 14 — Logging

### Почему это важно

Без централизованных логов отладка распределённых систем превращается в кошмар. Loki — лог-агрегатор в стиле Prometheus (лейблы вместо полного индекса), хорошо интегрируется с Grafana. Middle DevOps должен уметь настраивать сбор логов и писать LogQL-запросы.

Опираемся на:
- `[[DevOps/08_Monitoring/loki-and-logging]]`
- `[[DevOps/08_Monitoring/elk-stack]]`

---

## 1. Loki — архитектура

Из `[[DevOps/08_Monitoring/loki-and-logging]]`:

- **Promtail** — агент на хостах/в кластере, собирает логи и шлёт в Loki.
- **Loki** — хранилище логов (ingester, distributor, querier, storage).
- **Grafana** — UI для запросов LogQL.

Loki использует лейблы вместо полного индекса — дешёвое хранение, быстрые запросы по лейблам.

---

## 2. LogQL — базовые запросы

```logql
{app="api"}
{namespace="prod", pod=~"api-.*"}
{job="varlogs"} |= "ERROR"
{app="api"} |~ "timeout|deadline"
```

- `|=` — фильтр по подстроке.
- `|~` — фильтр по regex.

---

## 3. Парсинг полей

JSON:

```logql
{app="api"} | json | status >= 500
```

Pattern:

```logql
{app="nginx"} | pattern `<ip> - - <_> "<method> <path> <_>" <status> <size>`
| status =~ "5.."
```

---

## 4. Метрики из логов

```logql
sum(rate({app="api"} |= "ERROR" [5m]))
```

---

## 5. Promtail config

```yaml
scrape_configs:
  - job_name: varlogs
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

Для Kubernetes — DaemonSet, чтение `/var/log/containers/*.log`, лейблы из метаданных Pod.

---

## 6. ELK как альтернатива

Elasticsearch + Logstash + Kibana — полноценный индекс, более гибкий поиск, но тяжелее. См. `[[DevOps/08_Monitoring/elk-stack]]`.

---

## Что Middle DevOps должен знать (чек‑лист)

- [ ] Понимаю архитектуру Loki + Promtail (`[[DevOps/08_Monitoring/loki-and-logging]]`).
- [ ] Умею писать LogQL-запросы (фильтры, парсинг, метрики).
- [ ] Настраиваю Promtail для файлов и Kubernetes.
- [ ] Использую Grafana для просмотра логов.
