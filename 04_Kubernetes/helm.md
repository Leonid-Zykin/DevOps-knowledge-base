tags: [devops, helm]

## Helm — команды, структура чарта, values, templating

> ⚡ Tip: Helm = пакетный менеджер для Kubernetes. Чарты = набор манифестов + шаблоны, `values.yaml` = конфиг.

### Основные команды Helm 3

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo postgres                        # поиск чарта

helm install my-release bitnami/postgresql \
  --namespace db --create-namespace \
  --set auth.postgresPassword=secretpassword

helm list -A                                     # список релизов
helm status my-release -n db

helm upgrade my-release bitnami/postgresql \
  -n db \
  --set auth.postgresPassword=newsecret

helm rollback my-release 1 -n db                 # откат к ревизии 1

helm uninstall my-release -n db                  # удалить релиз
```

### Структура чарта

```text
mychart/
  Chart.yaml          # метаданные чарта
  values.yaml         # значения по умолчанию
  templates/          # шаблоны манифестов
  charts/             # зависимости
```

`Chart.yaml`:

```yaml
apiVersion: v2
name: mychart
description: A sample Helm chart
type: application
version: 0.1.0           # версия чарта
appVersion: "1.0.0"      # версия приложения
```

### Создание и рендеринг чарта

```bash
helm create mychart
ls mychart/templates

helm template mychart/ \
  --values myvalues.yaml \
  --namespace prod > rendered.yaml

kubectl apply -f rendered.yaml
```

`helm template` полезен для:

- локной отладки шаблонов;
- проверки итоговых манифестов в CI перед деплоем.

### Values и override'ы

```bash
helm install api ./charts/api \
  -f values-prod.yaml \
  --set image.tag=1.2.3 \
  --set resources.limits.cpu=500m
```

Приоритет:

1. значения из `--set`;
2. значения из `-f values-*.yaml`;
3. значения из `values.yaml` чарта.

Пример `values.yaml`:

```yaml
image:
  repository: registry.example.com/api
  tag: "latest"

replicaCount: 3

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

### Templating (Go templates)

Шаблон Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "mychart.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "mychart.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

Условные блоки:

```yaml
{{- if .Values.podAnnotations }}
  annotations:
    {{- toYaml .Values.podAnnotations | nindent 4 }}
{{- end }}
```

Циклы:

```yaml
env:
{{- range $key, $value := .Values.env }}
  - name: {{ $key }}
    value: {{ $value | quote }}
{{- end }}
```

### Проверка и lint

```bash
helm lint mychart/
helm template mychart/ --debug
```

### Зависимости чарта

`Chart.yaml`:

```yaml
dependencies:
  - name: redis
    version: 17.0.0
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

```bash
helm dependency update mychart/
```

### GitOps и Helm

- Helm часто используется совместно с Argo CD (`[[05_CI-CD/argocd]]`);
- рекомендуется:
  - версионировать values‑файлы;
  - не использовать `--set` для прод‑конфигов (только для ad‑hoc/dev).

Связанные заметки: `[[kubectl-cheatsheet]]`, `[[core-concepts]]`, `[[workloads]]`, `[[rbac-and-security]]`, `[[05_CI-CD/argocd]]`.

## Gotchas

- **Слишком много логики в шаблонах**  
  - **Проблема**: чарты становятся нечитаемыми, сложно отлаживать.  
  - **Решение**: максимально выносить вариативность в values, использовать простые шаблоны и функции `include`.

- **Использование `--set` для прод‑значений**  
  - **Проблема**: значения теряются, сложно воспроизвести конфигурацию.  
  - **Решение**: все прод‑override'ы хранить в values‑файлах и в Git; `--set` только для временных тестов.

- **Несогласованность версий чарта и приложения**  
  - **Проблема**: `appVersion` и `image.tag` не совпадают, сложно понять, что где деплоится.  
  - **Решение**: связать CI так, чтобы версия артефакта и `appVersion`/`image.tag` обновлялись атомарно.

> ⚠️ Warning: Ручной вызов `helm upgrade --install` с локальной машины против прод‑кластера без фиксации команд и значений в Git приводит к дрейфу конфигурации и «трудновоспроизводимым» релизам. В проде используйте GitOps (Argo CD/Flux) и храните как чарты, так и values в репозиториях.

