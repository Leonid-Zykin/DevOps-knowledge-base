## Module 12 — Mini-project: GitOps Pipeline to Kubernetes

### 1. Цель проекта

**Задача**: построить GitOps-пайплайн:

- CI собирает образ и пушит в registry;
- CI обновляет манифесты/values в Git (новый тег образа);
- Argo CD синхронизирует кластер с Git.

Оценочное время: **4–8 часов**.

---

### 2. Требования

1. **Git-репозиторий**:
   - Папка с Helm chart или Kustomize/plain manifests.
   - values-dev.yaml, values-prod.yaml (или аналог).
2. **CI**:
   - Build образа, push в registry.
   - Обновление образа в Git (image tag в values или kustomization).
   - Push изменений в Git (отдельная ветка или main).
3. **Argo CD**:
   - Application для приложения.
   - Automated sync с prune и selfHeal.
4. **Результат**:
   - Commit → CI build → push образа → update Git → Argo CD sync → приложение обновлено в кластере.

---

### 3. Пошаговый план

#### Шаг 1 — Репозиторий конфигурации

- Создайте репо `app-config` (или используйте монорепо с папкой `deploy/`).
- Добавьте Helm chart или manifests.
- values с placeholder для образа: `image.tag: "latest"` или от CI.

#### Шаг 2 — CI: build и push

- Job build: docker build, push в registry.
- Тег: `$CI_COMMIT_SHA` или `v1.2.3` при теге.

#### Шаг 3 — CI: update Git

- Job deploy-config: клонировать app-config, обновить image tag в values, commit, push.
- Использовать Git token (secret) для push.

#### Шаг 4 — Argo CD Application

- Создать Application, указывающий на app-config.
- path: charts/myapp или kustomize overlay.
- syncPolicy: automated, prune, selfHeal.

#### Шаг 5 — Проверка

- Сделать commit в app (код).
- CI собирает образ, обновляет Git.
- Argo CD синхронизирует, приложение обновляется в кластере.

---

### 4. Acceptance criteria

- [ ] CI собирает образ и пушит в registry
- [ ] CI обновляет конфигурацию в Git (image tag)
- [ ] Argo CD синхронизирует кластер с Git
- [ ] Изменение кода → новый образ → обновление в кластере без ручного kubectl/helm

---

### 5. Stretch goals

- Blue/Green или Canary через Argo CD/Ingress.
- Уведомления в Slack/Telegram при sync.
- SealedSecrets или SOPS для секретов в Git.

---

См. `[[DevOps/05_CI-CD/argocd]]`, `[[DevOps/05_CI-CD/concepts]]`, `[[DevOps/04_Kubernetes/helm]]`.
