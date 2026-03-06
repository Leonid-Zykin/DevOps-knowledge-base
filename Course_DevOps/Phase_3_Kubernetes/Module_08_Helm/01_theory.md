## Module 08 — Helm

### Почему это важно

Helm — пакетный менеджер для Kubernetes. Чарты позволяют переиспользовать конфигурации, параметризовать окружения и версионировать деплои. В сочетании с GitOps (Argo CD) Helm становится стандартом для деплоя приложений в K8s.

Опираемся на:
- `[[DevOps/04_Kubernetes/helm]]`
- `[[DevOps/04_Kubernetes/core-concepts]]`
- `[[DevOps/05_CI-CD/argocd]]`

---

## 1. Что такое Helm

- **Chart** — пакет манифестов + шаблоны (Go template) + values.
- **Release** — установленный экземпляр чарта в кластере.
- **Values** — параметры, переопределяющие дефолты чарта.

Helm 3 не использует Tiller; работает напрямую с API‑сервером K8s.

---

## 2. Основные команды

Из `[[DevOps/04_Kubernetes/helm]]`:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo postgres

helm install my-db bitnami/postgresql \
  --namespace db --create-namespace \
  --set auth.postgresPassword=secret

helm list -A
helm status my-db -n db

helm upgrade my-db bitnami/postgresql -n db --set auth.postgresPassword=newsecret
helm rollback my-db 1 -n db

helm uninstall my-db -n db
```

---

## 3. Структура чарта

```text
mychart/
  Chart.yaml          # метаданные
  values.yaml         # значения по умолчанию
  templates/          # манифесты с шаблонами
    deployment.yaml
    service.yaml
    _helpers.tpl
  charts/             # зависимости (subcharts)
```

`Chart.yaml`:

```yaml
apiVersion: v2
name: mychart
description: A sample Helm chart
type: application
version: 0.1.0
appVersion: "1.0.0"
```

---

## 4. Values и templating

В шаблонах используются переменные из `values.yaml` и `--set`:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

`values.yaml`:

```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.27"
```

Приоритет: `--set` > `-f values-prod.yaml` > `values.yaml`.

---

## 5. Функции и хелперы

В `_helpers.tpl`:

```yaml
{{- define "mychart.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
```

В шаблоне:

```yaml
metadata:
  name: {{ include "mychart.fullname" . }}
```

---

## 6. Условные блоки и циклы

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
...
{{- end }}
```

```yaml
env:
{{- range $key, $value := .Values.env }}
  - name: {{ $key }}
    value: {{ $value | quote }}
{{- end }}
```

---

## 7. helm template и lint

Проверка без установки:

```bash
helm template myrelease ./mychart --values values-prod.yaml
helm lint ./mychart
helm template myrelease ./mychart --debug
```

---

## 8. Зависимости (dependencies)

В `Chart.yaml`:

```yaml
dependencies:
  - name: redis
    version: "17.0.0"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

```bash
helm dependency update mychart/
```

---

## 9. GitOps и Helm

Argo CD может деплоить Helm‑чарты из Git:

```yaml
spec:
  source:
    repoURL: https://github.com/org/repo
    path: charts/myapp
    helm:
      valueFiles:
        - values-prod.yaml
```

Рекомендуется хранить values в Git, не использовать `--set` для прод‑конфигов. См. `[[DevOps/05_CI-CD/argocd]]`.

---

## Что Middle DevOps должен знать по Helm (чек‑лист)

- [ ] Умею устанавливать готовые чарты из репозиториев и переопределять values (`[[DevOps/04_Kubernetes/helm]]`).
- [ ] Понимаю структуру чарта (Chart.yaml, values.yaml, templates).
- [ ] Могу написать простой чарт с Deployment, Service, ConfigMap/Secret.
- [ ] Использую `helm template`, `helm lint` для проверки перед деплоем.
- [ ] Понимаю приоритет values (--set, -f, values.yaml) и не хардкожу прод‑конфиги в --set.
