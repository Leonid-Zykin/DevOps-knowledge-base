## Module 07 — Kubernetes Advanced

### Почему это важно

Продвинутые workloads (StatefulSet, DaemonSet, Job/CronJob), хранилища (PV/PVC/StorageClass), RBAC и NetworkPolicy — то, с чем DevOps сталкивается в реальных кластерах. Без этого вы не сможете корректно деплоить БД, агентов мониторинга и ограничивать доступ.

Опираемся на:
- `[[DevOps/04_Kubernetes/workloads]]`
- `[[DevOps/04_Kubernetes/storage]]`
- `[[DevOps/04_Kubernetes/rbac-and-security]]`
- `[[DevOps/04_Kubernetes/networking]]`
- `[[DevOps/04_Kubernetes/troubleshooting]]`

---

## 1. StatefulSet — stateful‑приложения

**Когда использовать**: БД, очереди с состоянием, приложения с постоянными идентификаторами.

Особенности:
- стабильные имена Pod’ов: `db-0`, `db-1`, …;
- порядок создания/удаления;
- связка с PersistentVolumeClaim через `volumeClaimTemplates`.

Пример из `[[DevOps/04_Kubernetes/workloads]]`:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi
```

Headless Service (`clusterIP: None`) даёт стабильные DNS‑имена: `postgres-0.postgres`, `postgres-1.postgres`.

---

## 2. DaemonSet — агент на каждой ноде

**Когда использовать**: node‑exporter, лог‑коллекторы, сетевые плагины.

DaemonSet гарантирует по одному Pod’у на каждой (подходящей) ноде.

Пример из `[[DevOps/04_Kubernetes/workloads]]`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:v1.8.2
          ports:
            - containerPort: 9100
```

---

## 3. Job и CronJob

**Job** — одноразовая задача (миграции, бэкапы, разовые скрипты):

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: app:migrations
          command: ["sh", "-c", "app migrate up"]
```

**CronJob** — периодический запуск Job’ов:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 3 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: tools:latest
              args: ["cleanup.sh"]
```

См. `[[DevOps/04_Kubernetes/workloads]]`.

---

## 4. PV, PVC, StorageClass

- **PersistentVolume (PV)** — кусок хранилища в кластере.
- **PersistentVolumeClaim (PVC)** — запрос на хранилище от Pod’а.
- **StorageClass** — провайдер и параметры динамического выделения.

Пример PVC из `[[DevOps/04_Kubernetes/storage]]`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-ssd
```

В Pod:

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
containers:
  - name: app
    volumeMounts:
      - name: data
        mountPath: /var/lib/app
```

AccessModes: `ReadWriteOnce` (RWO), `ReadOnlyMany` (ROX), `ReadWriteMany` (RWX). См. `[[DevOps/04_Kubernetes/storage]]`.

---

## 5. Resource requests и limits

Ограничение CPU/RAM для контейнеров:

```yaml
containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

- **requests** — гарантированный минимум, влияет на планирование.
- **limits** — верхняя граница; превышение memory → OOMKilled. См. `[[DevOps/04_Kubernetes/troubleshooting]]`.

---

## 6. RBAC: Role, RoleBinding, ServiceAccount

RBAC = кто (Subject) → что (Verb) → над чем (Resource).

**ServiceAccount**:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-sa
  namespace: prod
```

**Role** (в namespace):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prod
  name: api-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
```

**RoleBinding**:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-reader-binding
  namespace: prod
subjects:
  - kind: ServiceAccount
    name: api-sa
    namespace: prod
roleRef:
  kind: Role
  name: api-reader
  apiGroup: rbac.authorization.k8s.io
```

ClusterRole/ClusterRoleBinding — для кластерных ресурсов. См. `[[DevOps/04_Kubernetes/rbac-and-security]]`.

---

## 7. SecurityContext и Pod Security

Ограничения на уровне Pod/контейнера:

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
    - name: app
      securityContext:
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
```

Pod Security Admission (вместо устаревших PSP): уровни `privileged`, `baseline`, `restricted` на namespace. См. `[[DevOps/04_Kubernetes/rbac-and-security]]`.

---

## 8. NetworkPolicy

Ограничение трафика между Pod’ами (L3/L4). Требуется CNI с поддержкой (Calico, Cilium и т.п.).

Пример: разрешить доступ к `db` только от `api` (`[[DevOps/04_Kubernetes/networking]]`):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-from-api
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432
```

---

## 9. PodDisruptionBudget (PDB)

Защита от одновременного выбивания слишком большого числа реплик при drain/автоскейлинге:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: api
```

См. `[[DevOps/04_Kubernetes/workloads]]`.

---

## Что Middle DevOps должен знать по K8s Advanced (чек‑лист)

- [ ] Понимаю, когда использовать StatefulSet, DaemonSet, Job, CronJob (`[[DevOps/04_Kubernetes/workloads]]`).
- [ ] Умею создавать PVC и подключать их к Pod’ам/StatefulSet; понимаю StorageClass и AccessModes (`[[DevOps/04_Kubernetes/storage]]`).
- [ ] Задаю `resources.requests` и `limits` для контейнеров и понимаю последствия OOMKilled.
- [ ] Могу настроить ServiceAccount, Role, RoleBinding для ограниченного доступа (`[[DevOps/04_Kubernetes/rbac-and-security]]`).
- [ ] Понимаю базовую идею NetworkPolicy и SecurityContext (`[[DevOps/04_Kubernetes/networking]]`, `[[DevOps/04_Kubernetes/rbac-and-security]]`).
- [ ] Знаю, зачем нужен PodDisruptionBudget и как его задать.

Если чек‑лист выполняется — переходите к практике и мини‑проекту модуля.
