## Module 12 — CD & GitOps

### Почему это важно

GitOps — подход, при котором состояние инфраструктуры и приложений описывается в Git, а CD-система (Argo CD, Flux) синхронизирует кластер с репозиторием. Истина — в Git; ручные изменения в кластере считаются drift и откатываются. Middle DevOps должен понимать GitOps и уметь настраивать Argo CD.

Опираемся на:
- `[[DevOps/05_CI-CD/argocd]]`
- `[[DevOps/05_CI-CD/concepts]]`
- `[[DevOps/04_Kubernetes/helm]]`

---

## 1. GitOps — основные принципы

Из `[[DevOps/05_CI-CD/concepts]]`:

- **Истина в Git** — манифесты, Helm charts, Terraform в репозитории.
- **CI пушит в Git** — образы, теги, values обновляются в Git.
- **CD синхронизирует** — Argo CD/Flux приводит кластер к состоянию из Git.

---

## 2. Argo CD — базовые понятия

Из `[[DevOps/05_CI-CD/argocd]]`:

- **Application** — связь Git repo + path ↔ cluster/namespace.
- **Sync** — приведение кластера к состоянию из Git.
- **Health** — статус ресурсов (Healthy, Degraded, Progressing).
- **Drift** — расхождение между Git и кластером (ручные изменения).

---

## 3. Application (Helm)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: api
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/org/app-config.git'
    targetRevision: main
    path: charts/api
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 4. Sync policies

- **automated** — автоматическая синхронизация при изменениях в Git.
- **prune** — удалять ресурсы, убранные из Git.
- **selfHeal** — откатывать ручные изменения (drift) в кластере.

> [!warning] prune и selfHeal — мощные опции. Включайте с пониманием последствий.

---

## 5. App of Apps

Один root Application, который деплоит другие Applications:

```yaml
spec:
  source:
    path: apps/prod
  destination:
    namespace: argocd
```

Где `apps/prod` содержит манифесты Application для микросервисов.

---

## 6. Argo CD CLI

```bash
argocd login argocd.example.com
argocd app list
argocd app get api
argocd app sync api
argocd app diff api
argocd app rollback api 3
```

---

## 7. Стратегии деплоя

Из `[[DevOps/05_CI-CD/concepts]]`:

- **Rolling update** — постепенная замена подов (дефолт в K8s).
- **Blue/Green** — два окружения, переключение трафика.
- **Canary** — часть трафика на новую версию.

---

## 8. Секреты в GitOps

Не хранить секреты в открытом виде в Git. Использовать:
- SealedSecrets, SOPS, External Secrets;
- см. `[[DevOps/10_Security/secrets-management]]`.

---

## Что Middle DevOps должен знать по CD/GitOps (чек‑лист)

- [ ] Понимаю принципы GitOps (`[[DevOps/05_CI-CD/concepts]]`).
- [ ] Умею создавать и настраивать Argo CD Application (`[[DevOps/05_CI-CD/argocd]]`).
- [ ] Понимаю sync, prune, selfHeal.
- [ ] Знаю, как деплоить Helm-чарты через Argo CD (`[[DevOps/04_Kubernetes/helm]]`).
