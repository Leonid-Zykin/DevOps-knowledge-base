tags: [devops, loki-logging]

## Loki и логирование — LogQL, Promtail, паттерны

> ⚡ Tip: Loki — лог‑агрегатор в стиле Prometheus: лейблы вместо индексов, дешёвое хранение, особый язык запросов LogQL.

### Базовая архитектура

- **Promtail**: агент на хостах/кластере, собирает логи и шлёт в Loki.
- **Loki**: хранилище логов (ingester, distributor, querier, storage).
- **Grafana**: UI для запросов LogQL.

### Простой запрос LogQL

```logql
{app="api"}                               # все логи с лейблом app=api
{namespace="prod", pod=~"api-.*"}         # по namespace и pod regex
{job="varlogs"} |= "ERROR"                # фильтр по строке
{app="api"} |~ "timeout|deadline"         # regex-фильтр
```

### Парсинг полей

JSON:

```logql
{app="api"} | json | status >= 500
```

`pattern`:

```logql
{app="nginx"} | pattern `<ip> - - <_> "<method> <path> <_>" <status> <size>`
| status =~ "5.."
```

### Метрики из логов

```logql
sum(rate({app="api"} |= "ERROR" [5m]))
```

Histogram из логов:

```logql
sum by (le) (rate(
  {app="api"} | json | __error__="" | unwrap duration_seconds [5m]
))
```

### Promtail config (tail файлов)

`promtail-config.yaml`:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: varlogs
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

### Kubernetes интеграция

- DaemonSet Promtail, который читает контейнерные логи (`/var/log/containers/*.log`);
- лейблы из метаданных Pod/Namespace.

Связанные: `[[prometheus]]`, `[[grafana]]`, `[[elk-stack]]`, `[[11_Networking/troubleshooting]]`.

## Gotchas

- **Слишком детальные лейблы в логах**  
  - **Проблема**: каждый уникальный label (user_id, request_id) → новая stream, взрыв кардинальности.  
  - **Решение**: лейблы только для стабильных атрибутов (app, env, pod, level), переменные данные — в payload.

- **Парсинг JSON на горячую без лимитов**  
  - **Проблема**: тяжёлые `| json` на больших диапазонах времени.  
  - **Решение**: ограничивать диапазон, использовать сохранённые запросы/дашборды, по возможности нормализовать логи на стороне приложения.

- **Смешение структурированных и неструктурированных логов**  
  - **Проблема**: сложно писать универсальные запросы, часть логов теряется.  
  - **Решение**: стандартизировать формат логов (JSON с обязательными полями: level, msg, ts, service, trace_id).

> ⚠️ Warning: Логи могут содержать PII/секреты (токены, пароли, персональные данные). Прежде чем централизованно собирать и долго хранить логи, согласуйте политику маскирования/удаления, срок хранения и доступ с требованиями безопасности и комплаенса (`[[10_Security/secrets-management]]`, `[[10_Security/network-security]]`).

