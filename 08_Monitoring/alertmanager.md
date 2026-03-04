tags: [devops, alertmanager]

## Alertmanager — маршрутизация, приёмники, подавление

> ⚡ Tip: Alertmanager принимает алерты от Prometheus и отвечает за маршрутизацию, группировку, подавление и нотификации.

### Базовый `alertmanager.yml`

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  routes:
    - matchers:
        - severity="critical"
      receiver: 'pager'
      repeat_interval: 30m

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true

  - name: 'pager'
    pagerduty_configs:
      - routing_key: '<PAGERDUTY_KEY>'
```

### Matchers и routes

```yaml
routes:
  - matchers:
      - severity="warning"
      - team="backend"
    receiver: 'team-backend'
  - matchers:
      - env="prod"
      - severity="critical"
    receiver: 'oncall'
```

### Inhibition (подавление)

```yaml
inhibit_rules:
  - source_matchers:
      - severity="critical"
    target_matchers:
      - severity="warning"
    equal: ['alertname', 'cluster', 'service']
```

Когда есть critical‑алерт, соответствующий warning‑алерт подавляется.

### Silence

- В UI Alertmanager: Silences → New Silence.
- cli: `amtool`.

```bash
amtool silence add alertname="HighCPU" env="dev" --duration=2h
amtool silence query
```

Связанные: `[[prometheus]]`, `[[grafana]]`, `[[12_Soft-Skills-and-Processes/incident-response]]`.

## Gotchas

- **Нет группировки алертов**  
  - **Проблема**: при массовой проблеме получаете сотни уведомлений.  
  - **Решение**: использовать `group_by` (cluster, service, alertname), разумные `group_wait`/`group_interval`.

- **Нет inhibition между уровнями важности**  
  - **Проблема**: при critical‑инциденте продолжают приходить warning‑алармы.  
  - **Решение**: настраивать `inhibit_rules`, чтобы предупреждения глушились при наличии критичных алертов того же сервиса.

- **Мутящие silences без владельца и дедлайна**  
  - **Проблема**: кто‑то поставил «временную» тишину без expire, алерты перестали приходить.  
  - **Решение**: задавать `duration`, owner, comment, регулярно ревьюить silences.

> ⚠️ Warning: Маршрутизация critical‑алертов напрямую в общие каналы (Slack/Email) без выделенного on‑call канала/инструмента (PagerDuty, Opsgenie и т.п.) приводит к размытию ответственности и пропуску инцидентов. Критичные алерты должны иметь отдельный, чётко отработанный путь эскалации.

