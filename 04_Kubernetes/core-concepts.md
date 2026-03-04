tags: [devops, kubernetes-core]

## Kubernetes Core Concepts (Pod, Deployment, Service, Ingress, ConfigMap, Secret)

> ⚡ Tip: Почти всё в Kubernetes — это декларативные объекты (`kind`, `metadata`, `spec`) в API‑сервере, которыми управляет контроллер.

### Pod

Минимальная deploy‑единица (один или несколько контейнеров + shared network/storage).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Команды:

```bash
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

Используйте Pod напрямую в основном только для тестов/диагностики; для прод‑нагрузок — `Deployment`/`StatefulSet` (`[[workloads]]`).

### Deployment

Управляет ReplicaSet'ами и rollout'ами stateless приложений.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Ключевые поля:

- `spec.replicas` — количество pod'ов;
- `spec.selector` и `template.metadata.labels` — ДОЛЖНЫ совпадать.

```bash
kubectl get deploy
kubectl rollout status deployment nginx-deploy
kubectl set image deployment/nginx-deploy nginx=nginx:1.27.1
```

### Service

Абстракция над pod'ами; обеспечивает stable IP/DNS + балансировку.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

Типы:

- `ClusterIP` — доступ внутри кластера (по умолчанию);
- `NodePort` — порт на каждой ноде;
- `LoadBalancer` — внешний LB от облака.

```bash
kubectl get svc
kubectl describe svc nginx-svc
```

Смотрите `[[networking]]` для деталей Service/Ingress/NetworkPolicy.

### Ingress

HTTP/HTTPS вход в кластер через Ingress Controller (Nginx, Traefik и т.п.).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-svc
                port:
                  number: 80
```

```bash
kubectl get ingress
kubectl describe ingress nginx-ing
```

### ConfigMap

Неструктурированные конфиги (не секреты).

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "prod"
  LOG_LEVEL: "info"
  config.yml: |
    http:
      port: 8080
```

Монтирование как env:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

Монтирование как файл:

```yaml
volumes:
  - name: config
    configMap:
      name: app-config
containers:
  - name: app
    volumeMounts:
      - name: config
        mountPath: /app/config
```

### Secret

Базовое хранение секретов (base64‑кодирование, не шифрование).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0            # echo -n "secret" | base64
```

Env:

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

Файлы:

```yaml
volumes:
  - name: secret
    secret:
      secretName: app-secret
```

> ⚠️ Warning: Kubernetes Secrets хранятся в etcd без шифрования по умолчанию. Для прод‑кластеров включайте encryption at rest, RBAC, и лучше используйте внешние хранилища секретов (см. `[[10_Security/secrets-management]]`).

### Namespaces и multi-tenancy

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    env: dev
```

```bash
kubectl get ns
kubectl create ns dev
kubectl delete ns dev
```

Используйте:

- выделение окружений (dev/stage/prod);
- разделение команд/продуктов;
- ограничение через `ResourceQuota`, `LimitRange`, `RoleBinding` (`[[rbac-and-security]]`).

### Labels и selectors

- labels описывают объекты (`app=myapp`, `env=prod`);
- selectors используются сервисами, deployments, network policies.

```yaml
metadata:
  labels:
    app: myapp
    env: prod
spec:
  selector:
    matchLabels:
      app: myapp
```

```bash
kubectl get pods -l app=myapp
kubectl get svc -l env=prod
```

Связанные заметки: `[[workloads]]`, `[[networking]]`, `[[storage]]`, `[[rbac-and-security]]`, `[[helm]]`.

## Gotchas

- **Несоответствие selector'а и labels**  
  - **Проблема**: Service/Deployment не видят pod'ов (0 endpoints/replicas).  
  - **Решение**: проверять `spec.selector` и `metadata.labels` в Pod/Deployment/Service, использовать единый шаблон именования.

- **Хранение секретов в ConfigMap**  
  - **Проблема**: пароли/ключи попадают в логи и дампы ConfigMap'ов.  
  - **Решение**: всё чувствительное — только в Secret/внешних хранилищах, доступ ограничивать через RBAC.

- **Direct Pod deployment вместо контроллеров**  
  - **Проблема**: при падении Pod не пересоздаётся, нет rollout'ов.  
  - **Решение**: для прод‑нагрузок использовать `Deployment`, `StatefulSet`, `DaemonSet` (см. `[[workloads]]`).

> ⚠️ Warning: Ручное создание/редактирование объектов через `kubectl edit` в проде без фиксации в Git приводит к «дрейфу конфигурации». Используйте GitOps‑подходы (`[[05_CI-CD/argocd]]`, `[[helm]]`) и храните все манифесты в репозитории.

