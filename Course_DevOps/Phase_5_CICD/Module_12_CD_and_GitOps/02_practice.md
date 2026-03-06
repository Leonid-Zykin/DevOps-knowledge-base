## Module 12 — CD & GitOps Practice

Нужен Kubernetes-кластер (minikube, kind, облачный) с установленным Argo CD. Установка: `kubectl create namespace argocd && kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

Опирайтесь на `[[DevOps/05_CI-CD/argocd]]`, `[[DevOps/04_Kubernetes/helm]]`.

---

### Task 1 — Установка Argo CD

- **Goal**: Установить Argo CD в кластер.
- **Steps**:
  1. Создайте namespace argocd.
  2. Примените манифесты установки (см. документацию Argo CD).
  3. Дождитесь готовности подов: `kubectl get pods -n argocd`
  4. Получите пароль: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
- **Expected result**: Argo CD работает, доступен UI и CLI.

---

### Task 2 — Первое Application (plain manifests)

- **Goal**: Создать Application для простых манифестов.
- **Steps**:
  1. Создайте Git-репо с папкой `manifests/` (Deployment, Service).
  2. Создайте Application CR: source = repo + path `manifests/`, destination = namespace.
  3. Примените Application: `kubectl apply -f app.yaml`
  4. Проверьте sync в UI или `argocd app get <name>`
- **Expected result**: Ресурсы из Git развёрнуты в кластере.

---

### Task 3 — Application с Helm

- **Goal**: Деплой Helm-чарта через Argo CD.
- **Steps**:
  1. Используйте репо с Helm-чартом (свой или публичный).
  2. Создайте Application с `source.helm.valueFiles` или `source.helm.parameters`.
  3. Синхронизируйте и проверьте, что чарт установлен.
- **Expected result**: Helm-релиз управляется Argo CD.

---

### Task 4 — Automated sync

- **Goal**: Включить автоматическую синхронизацию.
- **Steps**:
  1. Добавьте в Application: `syncPolicy.automated.prune: true`, `selfHeal: true`
  2. Измените манифест в Git (например, replicas).
  3. Дождитесь автоматического sync (или проверьте интервал).
- **Expected result**: Кластер обновляется при изменении Git.

---

### Task 5 — Drift и selfHeal

- **Goal**: Понять, как Argo CD реагирует на ручные изменения.
- **Steps**:
  1. Вручную измените ресурс в кластере (kubectl edit).
  2. Проверьте статус Application — OutOfSync.
  3. При selfHeal: true Argo CD откатит изменение. Проверьте.
- **Expected result**: Понимание drift и selfHeal.

---

### Task 6 — argocd CLI

- **Goal**: Работать с Argo CD через CLI.
- **Steps**:
  1. Выполните `argocd app list`, `argocd app get <name>`
  2. Выполните `argocd app diff <name>` — посмотрите diff.
  3. Выполните `argocd app sync <name>` — принудительная синхронизация.
  4. Выполните `argocd app history <name>`, `argocd app rollback <name> <revision>`
- **Expected result**: Уверенная работа с CLI.

---

### Task 7 — App of Apps

- **Goal**: Создать root Application.
- **Steps**:
  1. Создайте папку в Git с несколькими Application-манифестами.
  2. Создайте root Application, указывающий на эту папку.
  3. Примените root — дочерние Applications должны создаться.
- **Expected result**: Один манифест разворачивает несколько приложений.

---

### Task 8 — Разные values для окружений

- **Goal**: Деплой в dev и prod с разными values.
- **Steps**:
  1. Создайте `values-dev.yaml` и `values-prod.yaml` в Git.
  2. Создайте два Application: один с values-dev (namespace dev), другой с values-prod (namespace prod).
  3. Проверьте, что конфигурации различаются.
- **Expected result**: Один чарт, разные окружения.

---

### Task 9 — Break it: prune

- **Goal**: Понять действие prune.
- **Steps**:
  1. Включите `prune: true`.
  2. Удалите ресурс из Git (например, ConfigMap).
  3. Синхронизируйте — ресурс должен удалиться из кластера.
  4. Восстановите ресурс в Git.
- **Expected result**: Понимание, что prune удаляет ресурсы, убранные из Git.

---

### Task 10 — Интеграция с CI

- **Goal**: CI обновляет Git, Argo CD синхронизирует.
- **Steps**:
  1. В CI после build добавьте шаг: обновить образ в Git (kustomize/helm values) и push.
  2. Argo CD должен подхватить изменение и задеплоить новую версию.
- **Expected result**: Полный цикл: commit → CI → push в Git → Argo CD sync.

---

## Итог

После практики вы умеете:
- устанавливать и настраивать Argo CD;
- создавать Application для manifests и Helm;
- работать с sync, prune, selfHeal;
- интегрировать CI с GitOps.
