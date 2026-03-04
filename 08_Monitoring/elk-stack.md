tags: [devops, elk]

## ELK Stack — Elasticsearch, Logstash, Kibana

> ⚡ Tip: ELK хорошо подходит для полнотекстового поиска и анализа логов, но требует аккуратной настройки ресурсов, шардирования и индексов.

### Elasticsearch базовые команды (API)

```bash
curl -u user:pass http://es:9200/
curl -u user:pass http://es:9200/_cat/indices?v
curl -u user:pass http://es:9200/_cluster/health?pretty
```

Создать индекс:

```bash
curl -X PUT -u user:pass http://es:9200/logs-2026.03.04
```

Поиск:

```bash
curl -X GET -u user:pass "http://es:9200/logs-*/_search?q=ERROR&pretty"
```

### Logstash pipeline (пример)

`logstash.conf`:

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  json {
    source => "message"
    skip_on_invalid_json => true
  }

  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }
}

output {
  elasticsearch {
    hosts => ["http://es:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

### Kibana

- Используйте Discover для интерактивного поиска логов;
- Visualize и Dashboards — для построения графиков;
- Dev Tools → Console — для Elasticsearch запросов.

Типичный DSL‑запрос:

```json
GET logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "app": "api" } },
        { "range": { "@timestamp": { "gte": "now-15m" } } }
      ],
      "must_not": [
        { "match": { "level": "debug" } }
      ]
    }
  }
}
```

Связанные: `[[loki-and-logging]]`, `[[prometheus]]`, `[[grafana]]`, `[[11_Networking/troubleshooting]]`.

## Gotchas

- **Чрезмерное количество индексов и шардов**  
  - **Проблема**: каждый индекс и шарда несут накладные расходы; тысячи индексов убивают кластер.  
  - **Решение**: использовать index lifecycle management (ILM), разумную ротацию и размер шард.

- **Отсутствие политики retention**  
  - **Проблема**: диски ES забиваются старыми логами.  
  - **Решение**: настраивать lifecycle/curator, удалять старые индексы, согласовывать сроки хранения.

- **Неструктурированные логи без парсинга**  
  - **Проблема**: сложно фильтровать по полям (status, latency, user_id).  
  - **Решение**: использовать filebeat/logstash ingest pipelines, логировать в JSON.

> ⚠️ Warning: Elasticsearch, особенно без ограничений на ingest, может быстро исчерпать ресурсы кластера (CPU, RAM, disk, heap). Всегда мониторьте метрики ES (`[[prometheus]]`, `[[grafana]]`) и тестируйте конфигурации на non‑prod окружении перед масштабным rollout'ом.

