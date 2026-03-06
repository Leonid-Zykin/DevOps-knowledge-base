## Phase 3 — Kubernetes (Weeks 7–10)

В этой фазе вы переходите от Docker/Compose к **оркестрации контейнеров в Kubernetes**:

- учитесь работать с Pod/Deployment/Service/Ingress;
- осваиваете продвинутые workloads (StatefulSet, DaemonSet, CronJob);
- разбираетесь с сетью, хранилищами, RBAC и Helm.

> [!info] Цель фазы
> К концу Phase 3 вы должны быть способны:
> - развернуть Dockerized‑приложение в локальном или облачном кластере Kubernetes;
> - масштабировать, обновлять и откатывать релизы;
> - описывать конфигурацию через Helm‑чарты.

---

## Структура Phase 3

- `Module_06_K8s_Basics/`
  - `01_theory.md` — базовые объекты и kubectl (`[[DevOps/04_Kubernetes/core-concepts]]`, `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`).
  - `02_practice.md` — деплой Pod/Deployment, Service, NodePort, масштабирование, rollback.
  - `03_project.md` — мини‑проект: деплой Dockerized‑приложения в minikube/kind.
  - `04_quiz.md` — проверка понимания.

- `Module_07_K8s_Advanced/`
  - `01_theory.md` — StatefulSet, DaemonSet, Jobs/CronJobs, RBAC, NetworkPolicy, resource limits (`[[DevOps/04_Kubernetes/workloads]]`, `[[DevOps/04_Kubernetes/storage]]`, `[[DevOps/04_Kubernetes/rbac-and-security]]`, `[[DevOps/04_Kubernetes/networking]]`).
  - `02_practice.md` — настройка stateful‑БД, DaemonSet‑агентов, CronJob’ов, ограничений ресурсов.
  - `03_project.md` — мини‑проект: stateful‑приложение (PostgreSQL + app).
  - `04_quiz.md` — проверка понимания.

- `Module_08_Helm/`
  - `01_theory.md` — Helm‑чарты, values, templating (`[[DevOps/04_Kubernetes/helm]]`).
  - `02_practice.md` — установка готовых чартов, написание собственного.
  - `03_project.md` — мини‑проект: упаковка своего приложения в Helm‑чарт и деплой в кластер.
  - `04_quiz.md` — проверка понимания.

---

## Ожидаемые результаты

К концу Phase 3 вы:

- **K8s Basics**
  - [ ] Понимаете ключевые объекты: Pod, Deployment, ReplicaSet, Service, Ingress, ConfigMap, Secret (`[[DevOps/04_Kubernetes/core-concepts]]`).
  - [ ] Уверенно используете `kubectl` для get/describe/apply/rollout/scale (`[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`).

- **K8s Advanced**
  - [ ] Умеете работать с StatefulSet, DaemonSet, Job, CronJob (`[[DevOps/04_Kubernetes/workloads]]`).
  - [ ] Разбираетесь в сети Kubernetes: Services, Ingress, NetworkPolicy (`[[DevOps/04_Kubernetes/networking]]`).
  - [ ] Управляете хранилищами через PV/PVC/StorageClass (`[[DevOps/04_Kubernetes/storage]]`).
  - [ ] Настраиваете базовый RBAC и security context (`[[DevOps/04_Kubernetes/rbac-and-security]]`).

- **Helm**
  - [ ] Устанавливаете готовые чарты и умеете их конфигурировать через values.
  - [ ] Пишете собственные чарт‑шаблоны с использованием базовых возможностей Helm (`[[DevOps/04_Kubernetes/helm]]`).

---

## Чек‑лист Phase 3

- [ ] Локальный кластер Kubernetes развёрнут (minikube/kind/k3d) и доступен через `kubectl`.
- [ ] Мини‑проект Module_06: ваше Docker‑приложение успешно развёрнуто в k8s, доступно через Service/Ingress.
- [ ] Мини‑проект Module_07: настроено stateful‑приложение (Postgres/другая БД + приложение) с PV/PVC.
- [ ] Мини‑проект Module_08: создан и задеплоен собственный Helm‑чарт приложения.
- [ ] Все квизы пройдены с результатом не менее 80%.

