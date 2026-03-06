# Final Project — Спецификация

## 1. Обзор

**Задача**: развернуть микросервисное приложение в облаке (или локально) с полным DevOps-стеком.

**Целевой уровень**: Middle DevOps.

---

## 2. Компоненты системы

### 2.1 Приложение

- **API** — веб-сервис (REST), например:
  - Простое API на Python/Node.js/Go с эндпоинтами: `/health`, `/api/items`.
  - Или готовое демо (например, demo-app с БД).
- **База данных** — PostgreSQL или Redis (StatefulSet или внешний managed DB).
- **Frontend** (опционально) — статика или SPA.

### 2.2 Инфраструктура

- **Terraform**:
  - VPC/сеть (или аналог для локального окружения).
  - Kubernetes-кластер (EKS, GKE, AKS или minikube/kind для локальной разработки).
  - Опционально: S3, RDS.
- **Модули**: network, compute, storage (разделение по доменам).

### 2.3 Kubernetes

- **Deployment** для API с replicas ≥ 2.
- **Service** (ClusterIP), **Ingress** (или LoadBalancer).
- **ConfigMap**, **Secret** (через SealedSecrets/SOPS).
- **Helm chart** для приложения.
- **RBAC** (опционально): ServiceAccount, Role, RoleBinding.

### 2.4 CI/CD

- **CI**:
  - Lint, test, build образа.
  - Push в registry (GHCR, GitLab Registry, ECR).
  - Trivy scan (container + dependencies).
- **CD (GitOps)**:
  - Argo CD Application для Helm chart.
  - Values в Git (dev/prod).
  - Automated sync с prune и selfHeal.

### 2.5 Мониторинг и логирование

- **Prometheus**: scrape метрик (node_exporter, приложение с /metrics).
- **Grafana**: дашборды (инфраструктура, приложение).
- **Alertmanager**: минимум 2 алерта (InstanceDown, HighCPU или аналог).
- **Loki + Promtail**: централизованные логи контейнеров.
- **Grafana**: панели логов.

### 2.6 Безопасность

- **Trivy** в CI (образы, зависимости).
- **Secrets**: SealedSecrets или SOPS, не в открытом виде в Git.
- **securityContext** в Pod: runAsNonRoot, readOnlyRootFilesystem, capabilities drop.
- **Non-root** образ (Dockerfile с USER).

---

## 3. Acceptance criteria

### Инфраструктура

- [ ] Terraform создаёт кластер/сеть одной командой.
- [ ] Используются минимум 2 модуля.
- [ ] Remote state (опционально).

### Kubernetes

- [ ] Приложение доступно через Ingress/LB.
- [ ] Реплики ≥ 2, health checks настроены.
- [ ] Helm chart параметризован (values dev/prod).

### CI/CD

- [ ] CI: lint, test, build, push, Trivy.
- [ ] Argo CD синхронизирует кластер с Git.
- [ ] Изменение кода → новый образ → обновление в кластере без ручного kubectl.

### Мониторинг

- [ ] Prometheus собирает метрики.
- [ ] Grafana: дашборды для инфраструктуры и приложения.
- [ ] Минимум 2 алерта.
- [ ] Loki: логи контейнеров в Grafana.

### Безопасность

- [ ] Trivy в CI, fail при HIGH/CRITICAL (или documented exceptions).
- [ ] Секреты не в открытом виде.
- [ ] securityContext применён к Pod'ам.

---

## 4. Окружения

- **dev** — минимальные ресурсы, 1 replica, для разработки.
- **prod** (или stage) — большие ресурсы, 2+ replicas, алерты.

---

## 5. Документация

- **README** в каждом репозитории:
  - Как развернуть (Terraform, Helm, Argo CD).
  - Как запустить CI локально (если применимо).
  - Описание алертов и как на них реагировать.
- **Архитектурная диаграмма** (Mermaid) в `architecture_diagram.md`.

---

## 6. Ограничения и допущения

- Можно использовать managed Kubernetes (EKS/GKE/AKS) или локальный (minikube, kind).
- Для облака — аккаунт с лимитами; после проекта — destroy ресурсов.
- Приложение может быть упрощённым (echo-API, demo-app), главное — полный стек.

---

## 7. Ссылки на базу знаний

- Terraform: `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/modules]]`
- Kubernetes: `[[DevOps/04_Kubernetes/core-concepts]]`, `[[DevOps/04_Kubernetes/helm]]`
- CI/CD: `[[DevOps/05_CI-CD/concepts]]`, `[[DevOps/05_CI-CD/argocd]]`
- Monitoring: `[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`, `[[DevOps/08_Monitoring/loki-and-logging]]`
- Security: `[[DevOps/10_Security/devsecops]]`, `[[DevOps/10_Security/secrets-management]]`, `[[DevOps/10_Security/container-security]]`
