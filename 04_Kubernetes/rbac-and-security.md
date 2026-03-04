tags: [devops, kubernetes-security]

## Kubernetes RBAC и безопасность (Role, ClusterRole, ServiceAccount, PSP/PodSecurity)

> ⚡ Tip: RBAC = кто (Subject) → что (Verb) → над чем (Resource). Всегда начинайте с принципа минимально необходимых прав.

### ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-sa
  namespace: prod
```

Использование в Pod:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: prod
spec:
  template:
    spec:
      serviceAccountName: api-sa
      containers:
        - name: api
          image: registry.example.com/api:1.2.3
```

### Role и RoleBinding (namespace‑ограниченные)

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

### ClusterRole и ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "nodes", "namespaces"]
    verbs: ["get", "list", "watch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-binding
subjects:
  - kind: User
    name: alice@example.com           # зависит от auth‑провайдера
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

Команды:

```bash
kubectl get sa -A
kubectl get roles -A
kubectl get clusterroles
kubectl get rolebindings -A
kubectl get clusterrolebindings
```

### Pod Security (PodSecurityAdmission вместо PSP)

(PSP устарели; используются уровни `privileged`, `baseline`, `restricted`.)

Пример namespace с `restricted`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: prod
  labels:
    pod-security.kubernetes.io/enforce: "restricted"
    pod-security.kubernetes.io/enforce-version: "latest"
```

Профиль `restricted` требует:

- non‑root контейнеров;
- ограничений capabilities;
- запрета hostPath/hostNetwork (по умолчанию).

### SecurityContext

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
      containers:
        - name: api
          image: myapp:latest
          securityContext:
            readOnlyRootFilesystem: true
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
```

### Проверка прав (kubectl auth can-i)

```bash
kubectl auth can-i get pods
kubectl auth can-i delete pods --as=alice@example.com -n prod
kubectl auth can-i '*' '*' --all-namespaces
```

### NetworkPolicy и TLS

- NetworkPolicy — ограничение L3/L4 (см. `[[networking]]`, `[[11_Networking/network-security]]`);
- TLS на Ingress/Service Mesh (см. `[[11_Networking/service-mesh]]`, `[[10_Security/network-security]]`).

Связанные: `[[core-concepts]]`, `[[workloads]]`, `[[10_Security/container-security]]`, `[[10_Security/secrets-management]]`.

## Gotchas

- **Использование `cluster-admin` для всех**  
  - **Проблема**: любой пользователь/SA может делать всё в кластере.  
  - **Решение**: создавать узкоспециализированные Role/ClusterRole, ограничивать доступ минимумом необходимых прав.

- **Pods, работающие от root и с привилегиями**  
  - **Проблема**: уязвимость в приложении даёт полный доступ к ноде.  
  - **Решение**: использовать `restricted` PodSecurity, `runAsNonRoot`, `readOnlyRootFilesystem`, отключать `allowPrivilegeEscalation`.

- **Непрозрачная схема аутентификации**  
  - **Проблема**: непонятно, какие реальные субъекты (User/Group/ServiceAccount) имеют какие права.  
  - **Решение**: документировать auth‑потоки, использовать OIDC/SSO, хранить артефакты RBAC в Git, регулярно ревьюить `ClusterRoleBinding`.

> ⚠️ Warning: Любые изменения ClusterRole/ClusterRoleBinding на прод‑кластере, особенно с широкими правами (`*` по ресурсам/действиям), могут дать неожиданно высокий уровень доступа CI‑пайплайнам, сервисам и пользователям. Перед применением внимательно проверяйте, кто будет Subject'ом и какие действия ему разрешены.

