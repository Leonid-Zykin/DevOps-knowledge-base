tags: [devops, kubernetes-workloads]

## Workloads: Deployments, StatefulSets, DaemonSets, Jobs, CronJobs

> ⚡ Tip: Stateless → `Deployment`, stateful (БД, очередь) → `StatefulSet`, фоновые агенты → `DaemonSet`, одноразовые задачи → `Job`/`CronJob`.

### Deployment — stateless приложения

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: registry.example.com/api:1.2.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

Команды:

```bash
kubectl get deploy
kubectl describe deploy api
kubectl rollout status deploy api
kubectl set image deploy/api api=registry.example.com/api:1.2.4
kubectl scale deploy api --replicas=5
```

### StatefulSet — stateful сервисы

Особенности:

- стабильные имена pod'ов: `db-0`, `db-1`, ...;
- связка с PersistentVolumeClaim (PVC);
- управляемые обновления по одному pod'у.

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
          ports:
            - containerPort: 5432
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

Сервис для доступа:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None              # headless service
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

### DaemonSet — агент на каждой ноде

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

Используется для:

- мониторинговых агентов;
- лог‑коллекторов;
- сетевых агентов на всех нодах.

### Job — одноразовая задача

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
          image: registry.example.com/app:migrations-1.2.3
          command: ["sh", "-c", "app migrate up"]
```

Команды:

```bash
kubectl get jobs
kubectl describe job db-migration
kubectl logs job/db-migration
```

### CronJob — периодические задачи

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 3 * * *"             # каждый день в 03:00
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 1
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: registry.example.com/tools:latest
              args: ["sh", "-c", "cleanup.sh"]
```

Команды:

```bash
kubectl get cronjobs
kubectl describe cronjob cleanup
kubectl get jobs --watch
```

### Strategies и PodDisruptionBudget

Deployment strategy:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

PDB:

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

PDB защищает от одновременного выпадения слишком большого числа pod'ов при drain/автоскейлинге.

Связанные заметки: `[[core-concepts]]`, `[[storage]]`, `[[networking]]`, `[[rbac-and-security]]`, `[[helm]]`.

## Gotchas

- **Использование Deployment вместо StatefulSet для БД**  
  - **Проблема**: потеря данных при пересоздании pod'ов, неправильная работа кластера БД.  
  - **Решение**: для БД и кластерных stateful‑сервисов использовать `StatefulSet` + headless Service + PV/PVC.

- **CronJob без контроля истории и backoff**  
  - **Проблема**: куча старых Job'ов, лавина перезапусков при ошибке.  
  - **Решение**: задавать `successfulJobsHistoryLimit`, `failedJobsHistoryLimit`, `backoffLimit` и мониторить статус (см. `[[08_Monitoring/prometheus]]`).

- **Отсутствие PodDisruptionBudget**  
  - **Проблема**: во время плановых работ (drain нод) можно выбить все реплики сервиса.  
  - **Решение**: определять PDB для критичных сервисов и согласовывать его с SLO (`[[12_Soft-Skills-and-Processes/sre-concepts]]`).

> ⚠️ Warning: Запуск миграций БД как обычных Pod'ов/Deployment'ов без контроля количества запусков и backoff может привести к многократному применению миграций, конфликтам схемы и простоям. Для этого используйте `Job`/`CronJob` с аккуратными настройками и мониторингом.

