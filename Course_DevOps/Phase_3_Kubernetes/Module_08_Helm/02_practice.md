## Module 08 — Helm Practice

Кластер и `helm` должны быть установлены. Опирайтесь на `[[DevOps/04_Kubernetes/helm]]`.

---

### Task 1 — Установка чарта из репозитория

- **Goal**: Установить готовый чарт (например, nginx или redis).
- **Steps**:
  1. Добавьте репозиторий Bitnami: `helm repo add bitnami https://charts.bitnami.com/bitnami`
  2. Обновите: `helm repo update`
  3. Установите: `helm install my-nginx bitnami/nginx -n default --create-namespace`
  4. Проверьте: `helm list`, `kubectl get pods`
- **Expected result**: Чарт установлен, Pod’ы запущены.

---

### Task 2 — Переопределение values через --set

- **Goal**: Изменить параметры при установке.
- **Steps**:
  1. Установите чарт с `--set service.type=NodePort`, `--set replicaCount=2`
  2. Проверьте, что Service имеет тип NodePort и реплик действительно 2
- **Expected result**: Values переопределены через CLI.

---

### Task 3 — Переопределение через values-файл

- **Goal**: Использовать отдельный YAML для values.
- **Steps**:
  1. Создайте `my-values.yaml` с нужными параметрами
  2. Установите: `helm install myapp bitnami/nginx -f my-values.yaml`
  3. Сравните с дефолтными values: `helm show values bitnami/nginx`
- **Expected result**: Конфигурация берётся из файла.

---

### Task 4 — helm create и базовая структура

- **Goal**: Создать свой чарт.
- **Steps**:
  1. Выполните: `helm create mychart`
  2. Изучите структуру: Chart.yaml, values.yaml, templates/
  3. Запустите: `helm lint mychart`
  4. Рендер без установки: `helm template test mychart`
- **Expected result**: Понимание структуры и работы шаблонов.

---

### Task 5 — Кастомизация Deployment в чарте

- **Goal**: Изменить образ и реплики через values.
- **Steps**:
  1. В `values.yaml` измените `image.repository`, `image.tag`, `replicaCount`
  2. В `templates/deployment.yaml` убедитесь, что используются `{{ .Values.image.repository }}` и т.п.
  3. Выполните `helm template test mychart` и проверьте итоговый YAML
- **Expected result**: Deployment параметризуется через values.

---

### Task 6 — Условный Ingress

- **Goal**: Добавить Ingress только если `ingress.enabled: true`.
- **Steps**:
  1. В values добавьте `ingress.enabled: false`
  2. В templates создайте ingress.yaml с `{{- if .Values.ingress.enabled }}`
  3. Проверьте: при `enabled: false` Ingress не должен появляться в `helm template`
- **Expected result**: Условный ресурс по флагу.

---

### Task 7 — Хелперы (define/include)

- **Goal**: Вынести общее имя в хелпер.
- **Steps**:
  1. В `_helpers.tpl` добавьте `define "mychart.fullname"`
  2. Используйте `{{ include "mychart.fullname" . }}` в metadata.name Deployment и Service
  3. Убедитесь, что имена согласованы
- **Expected result**: DRY через хелперы.

---

### Task 8 — helm upgrade и rollback

- **Goal**: Обновить релиз и откатить.
- **Steps**:
  1. Установите чарт: `helm install myapp ./mychart`
  2. Измените values (например, replicaCount) и выполните: `helm upgrade myapp ./mychart`
  3. Проверьте историю: `helm history myapp`
  4. Откатитесь: `helm rollback myapp 1`
- **Expected result**: Уверенная работа с upgrade/rollback.

---

### Task 9 — Зависимости (dependencies)

- **Goal**: Добавить subchart.
- **Steps**:
  1. В Chart.yaml добавьте dependency (например, redis из Bitnami)
  2. Выполните: `helm dependency update mychart`
  3. Проверьте каталог `charts/`
  4. Установите чарт и убедитесь, что dependency тоже развернулась
- **Expected result**: Понимание зависимостей чартов.

---

### Task 10 — Интеграция с вашим приложением

- **Goal**: Упаковать ваше приложение из Phase 2/3 в чарт.
- **Steps**:
  1. Создайте чарт с Deployment, Service, ConfigMap, Secret (шаблонизированными)
  2. Вынесите образ, реплики, env в values
  3. Установите в кластер и проверьте работу
- **Expected result**: Ваше приложение деплоится через Helm.

---

## Итог

После практики вы умеете:
- устанавливать и настраивать готовые чарты;
- создавать свои чарты с шаблонами и values;
- использовать upgrade/rollback и зависимости;
- подготавливать чарт для GitOps (Argo CD).
