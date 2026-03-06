## Module 06 — Kubernetes Basics

### Почему это важно

Kubernetes — стандартный способ запускать контейнеры в продакшене. DevOps‑инженер должен понимать:

- какие есть базовые объекты (Pod, Deployment, Service, Ingress, ConfigMap, Secret, Namespace);
- как ими управлять через `kubectl`;
- как деплоить и обновлять приложения без простоя.

Мы опираемся на:

- `[[DevOps/04_Kubernetes/core-concepts]]`
- `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`
- `[[DevOps/04_Kubernetes/troubleshooting]]`

---

## 1. Архитектура на верхнем уровне

Упрощённо:

- **API server** — точка входа для всех операций (kubectl, controllers);
- **etcd** — хранилище состояния кластера;
- **kube-scheduler** — выбирает ноды для Pod’ов;
- **kubelet** — агент на ноде, следит за Pod’ами;
- **kube-proxy/CNI** — сеть.

> [!info] Главное
> Всё в Kubernetes — это **объекты** (YAML‑ресурсы) в API‑сервере. Вы описываете **состояние**, а контроллеры доводят реальность до этого состояния.

---

## 2. Pod — минимальная единица деплоя

Из `[[DevOps/04_Kubernetes/core-concepts]]`:

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

- Pod может содержать несколько контейнеров, разделяющих сеть и volumes;
- обычно Pod’ы создаются контроллерами (Deployment/StatefulSet), а не вручную.

Основные команды:

```bash
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl exec -it nginx-pod -- bash
```

---

## 3. Deployment и ReplicaSet

Deployment управляет ReplicaSet’ами и rollout’ами stateless‑приложений:

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

Ключевые моменты:

- `spec.selector.matchLabels` **должен совпадать** с `template.metadata.labels`;
- Deployment отвечает за:
  - количество реплик;
  - стратегию обновлений;
  - откаты (`rollout undo`).

Команды (`[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`):

```bash
kubectl get deploy
kubectl describe deploy nginx-deploy
kubectl rollout status deploy nginx-deploy
kubectl set image deploy/nginx-deploy nginx=nginx:1.27.1
kubectl scale deploy nginx-deploy --replicas=5
kubectl rollout undo deploy nginx-deploy
```

---

## 4. Service — стабильная точка доступа

Service обеспечивает:

- стабильное имя/DNS и виртуальный IP;
- балансировку по Pod’ам по label‑selector’у.

Пример (`[[DevOps/04_Kubernetes/core-concepts]]`):

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

- `ClusterIP` — доступ только внутри кластера;
- `NodePort` — порт на нодах кластера;
- `LoadBalancer` — внешний LB в облаке.

Проверка:

```bash
kubectl get svc
kubectl get endpoints nginx-svc
```

Сеть детально — `[[DevOps/04_Kubernetes/networking]]`.

---

## 5. Ingress — входящий HTTP/HTTPS‑трафик

Ingress описывает правила роутинга HTTP/HTTPS на Services. Требуется Ingress Controller (nginx, Traefik, etc).

Пример (`[[DevOps/04_Kubernetes/core-concepts]]`):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ing
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

Команды:

```bash
kubectl get ingress
kubectl describe ingress nginx-ing
```

---

## 6. ConfigMap и Secret

Конфиги и секреты выносите в отдельные объекты (`[[DevOps/04_Kubernetes/core-concepts]]`).

**ConfigMap**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "prod"
  LOG_LEVEL: "info"
```

**Secret**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0  # "secret" base64
```

Использование в Pod:

```yaml
envFrom:
  - configMapRef:
      name: app-config
  - secretRef:
      name: app-secret
```

> [!warning] Секреты в Kubernetes
> Kubernetes Secrets базово только base64‑кодируются. Для прода нужен encryption at rest, RBAC и (лучше) внешние секрет‑сервисы (`[[DevOps/10_Security/secrets-management]]`).

---

## 7. Namespaces и labels

Namespaces:

- логически разделяют окружения и команды;
- используются для изоляции прав, квот и сетевых политик.

Пример:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    env: dev
```

Labels:

- описывают объекты (`app=myapp`, `env=prod`);
- используются в selectors (Deployment, Service, NetworkPolicy).

```bash
kubectl get pods -l app=myapp
kubectl get svc -l env=prod
```

---

## 8. kubectl: базовые паттерны

Из `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`:

```bash
kubectl get pods -o wide
kubectl describe pod mypod
kubectl logs mypod
kubectl exec -it mypod -- bash

kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl diff -f manifest.yaml
kubectl apply -f k8s/  # каталог манифестов
```

Dry‑run:

```bash
kubectl apply -f deploy.yaml --dry-run=client -o yaml
kubectl diff -f deploy.yaml
```

> [!tip] GitOps
> В продакшене манифесты обычно хранятся в Git и применяются через Argo CD/Flux (`[[DevOps/05_CI-CD/argocd]]`, `[[DevOps/04_Kubernetes/helm]]`).

---

## 9. Troubleshooting на базовом уровне

Из `[[DevOps/04_Kubernetes/troubleshooting]]`:

1. **Pod не запускается / CrashLoopBackOff**:
   - `kubectl describe pod`;
   - `kubectl logs pod --previous`.
2. **Pod Pending**:
   - `kubectl describe pod` → нет ресурсов/нода не подходит/PVC Pending.
3. **Service не доступен**:
   - `kubectl get endpoints`;
   - `kubectl exec debug-pod -- curl -v http://svc-name`.

---

## Что Middle DevOps должен знать по K8s Basics (чек‑лист)

- [ ] Понимаю разницу между Pod и Deployment и почти никогда не деплою Pod напрямую.
- [ ] Могу создать Deployment + Service + (опционально) Ingress для простого web‑приложения.
- [ ] Уверенно использую `kubectl get/describe/logs/exec` для диагностики.
- [ ] Знаю, как вынести конфигурацию в ConfigMap и Secret и смонтировать их в Pod.
- [ ] Понимаю назначение Namespaces и умею работать с ними.
- [ ] Знаю базовый процесс troubleshooting CrashLoopBackOff и Pending Pod’ов (`[[DevOps/04_Kubernetes/troubleshooting]]`).

Если чек‑лист выполняется — переходите к практическим заданиям и мини‑проекту для этого модуля.

