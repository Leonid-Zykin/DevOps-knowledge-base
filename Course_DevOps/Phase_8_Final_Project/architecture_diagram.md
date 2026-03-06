# Final Project — Архитектурная диаграмма

## Общая схема (Mermaid)

```mermaid
flowchart TB
  subgraph External
    User[Пользователь]
    Git[Git Repository]
    Registry[Container Registry]
  end

  subgraph CI["CI Pipeline"]
    Lint[Lint]
    Test[Test]
    Build[Build Image]
    Trivy[Trivy Scan]
    Push[Push to Registry]
    UpdateGit[Update Git<br/>image tag]
    Lint --> Test --> Build --> Trivy --> Push --> UpdateGit
  end

  subgraph GitOps["GitOps (Argo CD)"]
    ArgoCD[Argo CD]
    AppConfig[App Config Repo<br/>Helm values, manifests]
    ArgoCD -->|sync| AppConfig
  end

  subgraph Infra["Infrastructure (Terraform)"]
    TF[ Terraform ]
    VPC[VPC / Network]
    K8s[Kubernetes Cluster]
    TF --> VPC --> K8s
  end

  subgraph K8s["Kubernetes Cluster"]
    Ingress[Ingress / LB]
    API[API Deployment]
    DB[(PostgreSQL/Redis)]
    API --> DB
    Ingress --> API
  end

  subgraph Monitoring["Monitoring Stack"]
    Prometheus[Prometheus]
    Grafana[Grafana]
    Loki[Loki]
    Promtail[Promtail]
    Prometheus --> Grafana
    Loki --> Grafana
    Promtail --> Loki
  end

  User --> Ingress
  Git --> CI
  Push --> Registry
  UpdateGit --> Git
  AppConfig --> ArgoCD
  ArgoCD -->|deploy| K8s
  K8s --> Prometheus
  K8s --> Promtail
  Infra --> K8s
```

---

## Поток данных

1. **Разработка**: коммит в Git → CI (lint, test, build, Trivy, push) → обновление образа в Git (values/chart).
2. **Деплой**: Argo CD синхронизирует кластер с Git → приложение обновляется.
3. **Трафик**: User → Ingress → API → DB.
4. **Мониторинг**: Prometheus scrape метрик, Promtail собирает логи → Grafana.

---

## Компоненты по слоям

| Слой | Компоненты |
|------|------------|
| **Infra** | Terraform, VPC, K8s cluster |
| **App** | API (Deployment), DB (StatefulSet/managed) |
| **Networking** | Ingress, Service |
| **CI/CD** | GitHub Actions / GitLab CI, Argo CD |
| **Monitoring** | Prometheus, Grafana, Loki, Promtail, Alertmanager |
| **Security** | Trivy, SealedSecrets/SOPS, securityContext |
