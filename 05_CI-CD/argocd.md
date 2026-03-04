tags: [devops, argocd]

## Argo CD — GitOps, App of Apps, sync

> ⚡ Tip: Argo CD синхронизирует состояние Kubernetes‑кластера с Git‑репозиторием. Любые ручные изменения в кластере считаются «drift» и могут быть автоматически откатаны.

### Базовые понятия

- **Application**: связь Git repo ↔ путь ↔ cluster/namespace.
- **Sync**: приведение кластера к состоянию из Git.
- **Health**: статус ресурса (Healthy, Degraded и т.д.).
- **Project**: группировка приложений и политики доступа.

### Пример Application (Helm)

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
    syncOptions:
      - CreateNamespace=true
```

Связанные: `[[04_Kubernetes/helm]]`, `[[04_Kubernetes/core-concepts]]`.

### App of Apps pattern

`root` Application, описывающий другие Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/org/cluster-config.git'
    targetRevision: main
    path: apps/prod
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Где `apps/prod` содержит набор `Application`‑манифестов для микросервисов.

### CLI (`argocd`)

```bash
argocd login argocd.example.com
argocd app list
argocd app get api
argocd app sync api
argocd app diff api
argocd app history api
argocd app rollback api 3
```

### Sync policies

```yaml
syncPolicy:
  automated:
    prune: true         # удалять ресурсы, убранные из Git
    selfHeal: true      # откатывать drift
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

### Best practices

- все манифесты/Helm‑values в Git;
- только Argo CD (а не люди/CI) имеет право применять манифесты на кластер;
- использовать проекты и RBAC для разделения команд/систем;
- мониторить `OutOfSync`/`Degraded` через Prometheus/Grafana (`[[08_Monitoring/prometheus]]`, `[[08_Monitoring/grafana]]`).

Связанные: `[[05_CI-CD/concepts]]`, `[[04_Kubernetes/troubleshooting]]`, `[[10_Security/secrets-management]]`.

## Gotchas

- **Ручные `kubectl apply` поверх Argo CD**  
  - **Проблема**: Argo CD считает изменения drift'ом и откатывает их или постоянно показывает OutOfSync.  
  - **Решение**: правки делать только через Git и sync Argo CD; ручные фиксы → отдельные MR/PR.

- **Автоматический `prune` без понимания последствий**  
  - **Проблема**: ресурсы, убранные из Git, автоматически удаляются из кластера (включая PV‑зависимости).  
  - **Решение**: включать `prune` постепенно, тщательно ревьюить diff перед sync.

- **Хранение секретов в Git в открытом виде**  
  - **Проблема**: GitOps делает всё, включая секреты, частью репо.  
  - **Решение**: использовать SealedSecrets, SOPS, внешние хранилища (`[[10_Security/secrets-management]]`), а не голые Secret‑манифесты.

> ⚠️ Warning: Неправильная настройка доступа к Argo CD (UI/API) и отсутствующие ограничения на sync (например, возможность любому пользователю сделать sync в prod) могут привести к случайным или злонамеренным деплоям. Используйте RBAC, SSO и разделение проектов/кластеров.

