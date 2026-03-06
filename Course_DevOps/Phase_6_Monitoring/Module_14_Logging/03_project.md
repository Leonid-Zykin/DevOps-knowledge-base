## Module 14 — Mini-project: Centralized Logging for Kubernetes

### 1. Цель проекта

**Задача**: развернуть централизованное логирование для Kubernetes:

- Loki для хранения;
- Promtail для сбора логов контейнеров;
- Grafana для просмотра и поиска.

Оценочное время: **4–6 часов**.

---

### 2. Требования

1. **Loki**:
   - Развёрнут в кластере (Deployment или Helm).
   - Хранение: local или S3/GCS (опционально).
2. **Promtail**:
   - DaemonSet на всех нодах.
   - Чтение `/var/log/containers/*.log`.
   - Лейблы: namespace, pod, container_name, app.
3. **Grafana**:
   - Loki datasource.
   - Дашборд с панелями логов по namespace, pod.
   - Переменные для фильтрации.
4. **Приложение**:
   - Развернуть тестовое приложение, генерирующее логи.
   - Проверить поиск по логам.

---

### 3. Пошаговый план

#### Шаг 1 — Loki

- Установите Loki (Helm: `loki` или `loki-stack`).
- Проверьте, что Loki принимает логи.

#### Шаг 2 — Promtail

- Установите Promtail как DaemonSet.
- Конфиг: kubernetes_sd_configs для обнаружения подов, pipeline для парсинга путей.
- Лейблы из `__meta_kubernetes_namespace`, `__meta_kubernetes_pod_name` и т.д.

#### Шаг 3 — Grafana

- Добавьте Loki datasource.
- Создайте дашборд: панели логов, переменные (namespace, pod).

#### Шаг 4 — Тестовое приложение

- Задеплойте приложение с логами (например, nginx или простое API).
- Сгенерируйте ошибки и успешные запросы.
- Проверьте поиск: `{namespace="default"} |= "error"`.

#### Шаг 5 — Документация

- README: как развернуть стек, примеры запросов LogQL.

---

### 4. Acceptance criteria

- [ ] Loki и Promtail развёрнуты в K8s
- [ ] Логи контейнеров попадают в Loki
- [ ] Grafana отображает логи с фильтрацией по namespace/pod
- [ ] Можно найти логи по ключевым словам (ERROR, exception и т.п.)

---

### 5. Stretch goals

- Retention policy для Loki.
- Метрики из логов (rate ошибок) в отдельной панели.
- Интеграция с Alertmanager (алерты на основе LogQL).

---

См. `[[DevOps/08_Monitoring/loki-and-logging]]`, `[[DevOps/08_Monitoring/elk-stack]]`.
