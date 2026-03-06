# Глоссарий — ключевые термины курса

Краткие определения для быстрого поиска.

---

## A

- **Ansible** — инструмент идемпотентной конфигурации серверов (playbooks, roles, inventory).
- **Argo CD** — GitOps-инструмент для синхронизации Kubernetes с Git.
- **Artifact** — результат сборки (образ, бинарь, архив).
- **Alertmanager** — компонент Prometheus для маршрутизации и отправки уведомлений об алертах.

---

## B

- **Blue/Green** — стратегия деплоя: два окружения, переключение трафика.
- **Backend (Terraform)** — хранилище state (S3, GCS, Azure).

---

## C

- **CI (Continuous Integration)** — автоматическая сборка и тестирование при коммите.
- **CD (Continuous Delivery/Deployment)** — доставка/деплой до прода.
- **ConfigMap** — объект K8s для несекретной конфигурации.
- **Counter** — тип метрики Prometheus, монотонно растущий.
- **Canary** — стратегия деплоя: часть трафика на новую версию.

---

## D

- **DAST** — динамическое тестирование безопасности (запуск приложения).
- **Data source (Terraform)** — чтение существующих ресурсов без управления.
- **Deployment** — объект K8s для stateless приложений.
- **DevSecOps** — встраивание безопасности в CI/CD.
- **Drift** — расхождение между Git и кластером (ручные изменения).
- **Dockerfile** — инструкции для сборки образа.

---

## E

- **Exporter** — сервис, отдающий метрики в формате Prometheus (node_exporter и т.д.).

---

## G

- **Gauge** — тип метрики Prometheus, может расти и падать.
- **GitOps** — подход: истина в Git, CD синхронизирует кластер.
- **Grafana** — визуализация метрик и логов.
- **group_vars** — переменные Ansible для группы хостов.

---

## H

- **Handler (Ansible)** — задача, выполняемая при notify и changed.
- **Helm** — пакетный менеджер для Kubernetes (charts, releases).
- **Histogram** — тип метрики Prometheus (bucket'ы, sum, count).

---

## I

- **Idempotency** — повторный запуск не меняет уже настроенное состояние.
- **Ingress** — объект K8s для маршрутизации HTTP(S) трафика.
- **Inventory** — список хостов Ansible.

---

## J

- **Job (Kubernetes)** — одноразовое выполнение задач.
- **Job (CI)** — единица работы в пайплайне (GitHub Actions, GitLab CI).

---

## K

- **kubectl** — CLI для Kubernetes.
- **Kubernetes (K8s)** — оркестратор контейнеров.

---

## L

- **Liveness probe** — проверка «жив ли» контейнер.
- **Loki** — лог-агрегатор в стиле Prometheus (лейблы).
- **LogQL** — язык запросов Loki.
- **Locals (Terraform)** — вычисляемые значения в конфигурации.

---

## M

- **Module (Terraform)** — переиспользуемый набор ресурсов.
- **Metrics** — числовые показатели (Prometheus).

---

## N

- **Namespace** — изоляция ресурсов в Kubernetes.
- **Node** — рабочая машина в кластере K8s.

---

## O

- **Output (Terraform)** — экспорт значения из модуля/конфигурации.
- **OutOfSync** — состояние Argo CD: кластер не соответствует Git.

---

## P

- **Pipeline** — последовательность стадий CI/CD.
- **Playbook** — YAML-файл с задачами Ansible.
- **Pod** — минимальная единица деплоя в K8s (один или несколько контейнеров).
- **Prometheus** — система сбора и хранения метрик.
- **PromQL** — язык запросов Prometheus.
- **Promtail** — агент сбора логов для Loki.
- **prune** — удаление из кластера ресурсов, убранных из Git (Argo CD).

---

## R

- **Rate** — функция Prometheus: скорость прироста метрики.
- **Readiness probe** — проверка готовности пода принимать трафик.
- **Release (Helm)** — установленный экземпляр чарта.
- **Resource (Terraform)** — объект инфраструктуры, которым управляет Terraform.
- **Role (Ansible)** — переиспользуемый набор tasks, handlers, templates.
- **Rolling update** — постепенная замена подов без простоя.

---

## S

- **SAST** — статический анализ кода на уязвимости.
- **Scrape** — сбор метрик Prometheus с targets.
- **Secret** — объект K8s для чувствительных данных.
- **SealedSecrets** — зашифрованный Secret для Git.
- **selfHeal** — откат drift в кластере к состоянию из Git (Argo CD).
- **securityContext** — настройки безопасности контейнера (runAsNonRoot и т.д.).
- **Service** — объект K8s для доступа к подам (ClusterIP, NodePort, LoadBalancer).
- **SOPS** — шифрование файлов с секретами (PGP, KMS).
- **State (Terraform)** — текущее состояние инфраструктуры.
- **StatefulSet** — объект K8s для stateful приложений.
- **Sync** — приведение кластера к состоянию из Git (Argo CD).

---

## T

- **Task (Ansible)** — единица работы в playbook.
- **Terraform** — инструмент IaC (Infrastructure as Code).
- **Trivy** — сканер уязвимостей (образы, FS, конфиги).

---

## V

- **Values (Helm)** — параметры чарта (values.yaml, --set).
- **Variable (Terraform)** — входной параметр конфигурации.
- **Vault (Ansible)** — шифрование секретов.
- **Vault (HashiCorp)** — хранилище секретов.
- **Volume** — хранилище данных в Docker/K8s.

---

## W

- **Workflow** — описание пайплайна в GitHub Actions.
- **Workspace (Terraform)** — логическое окружение (dev, prod).
