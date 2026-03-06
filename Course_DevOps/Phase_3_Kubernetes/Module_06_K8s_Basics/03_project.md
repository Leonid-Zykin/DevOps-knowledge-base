## Module 06 — Mini‑project: Deploy Your Dockerized App to Kubernetes

### 1. Цель проекта

**Задача**: взять Docker‑приложение из Phase 2 (Module_04/05) и:

- задеплоить его в локальный кластер Kubernetes (minikube/kind);
- настроить Service (ClusterIP/NodePort) и, по возможности, Ingress;
- использовать ConfigMap/Secret для конфигурации;
- настроить грамотный rollout и уметь его откатывать.

Оценочное время: **4–8 часов**.

---

### 2. Исходные условия

У вас уже должно быть:

- рабочее приложение, упакованное в Docker‑образ (см. `Phase_2/Module_04_Docker/03_project.md`);
- локальный кластер (minikube/kind) и настроенный `kubectl`.

Образ должен быть доступен в кластере:

- либо через локальный registry;
- либо через Docker Hub/другой публичный registry;
- либо через команды импортирования образа в minikube/kind.

---

### 3. Требования к итоговому деплою

1. **Namespace**:
   - отдельный namespace для приложения (например, `demo-k8s-basics`).
2. **Deployment**:
   - как минимум 2 реплики;
   - image — ваш реальный образ (не `nginx`);
   - ресурсы: заданы `requests` и `limits` для CPU/RAM.
3. **Service**:
   - тип ClusterIP для внутреннего доступа;
   - опционально NodePort или Ingress для внешнего доступа.
4. **ConfigMap+Secret**:
   - конфиг (APP_ENV, LOG_LEVEL) в ConfigMap;
   - чувствительные данные (например, DB_PASSWORD/API_KEY) в Secret.
5. **Rollout**:
   - обновление образа до новой версии;
   - откат через `kubectl rollout undo`.

---

### 4. Пошаговый план

#### Шаг 1 — Namespace и базовая структура

1. Создайте namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo-k8s-basics
  labels:
    env: demo
```

2. Примените:

```bash
kubectl apply -f namespace.yaml
kubectl config set-context --current --namespace=demo-k8s-basics
```

3. Создайте директорию `k8s/` со следующими файлами:

```text
k8s/
  namespace.yaml
  deployment.yaml
  service.yaml
  configmap.yaml
  secret.yaml
  ingress.yaml    # опционально
```

#### Шаг 2 — ConfigMap и Secret

Пример `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "dev"
  LOG_LEVEL: "info"
```

Пример `secret.yaml` (пароль закодирован base64):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0  # echo -n "secret" | base64
```

#### Шаг 3 — Deployment вашего приложения

`deployment.yaml` (упрощённый пример):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: your-registry/your-app:1.0.0
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef:
                name: app-config
            - secretRef:
                name: app-secret
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

Примените:

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl get pods
kubectl describe deploy app-deploy
```

#### Шаг 4 — Service (ClusterIP и NodePort/Ingress)

`service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-svc
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8000
      protocol: TCP
  type: ClusterIP
```

Примените:

```bash
kubectl apply -f k8s/service.yaml
kubectl get svc
kubectl get endpoints app-svc
```

Для доступа снаружи выберите один вариант:

- NodePort:

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30080
```

- или Ingress (для minikube с включённым Ingress Controller):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ing
spec:
  ingressClassName: nginx
  rules:
    - host: app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-svc
                port:
                  number: 80
```

#### Шаг 5 — Проверка и отладка

- Проверка изнутри кластера:

```bash
kubectl run debug -it --rm --image=busybox:1.36 --restart=Never -- sh
/ # wget -qO- http://app-svc
```

- Проверка снаружи:

```bash
kubectl port-forward svc/app-svc 8080:80
curl http://127.0.0.1:8080/health
```

Используйте `kubectl describe`, `kubectl logs`, `kubectl get events` при любых проблемах (см. `[[DevOps/04_Kubernetes/troubleshooting]]`).

#### Шаг 6 — Обновление версии и rollback

1. Обновите образ до новой версии (например, `1.1.0`) в `deployment.yaml` или через команду:

```bash
kubectl set image deploy/app-deploy app=your-registry/your-app:1.1.0
kubectl rollout status deploy/app-deploy
```

2. Если релиз «сломал» приложение — выполните откат:

```bash
kubectl rollout history deploy/app-deploy
kubectl rollout undo deploy/app-deploy
```

---

### 5. Acceptance criteria

Мини‑проект считается завершённым, если:

1. **Namespace и объекты**:
   - все ресурсы (Deployment, Service, ConfigMap, Secret, опционально Ingress) живут в отдельном namespace;
   - `kubectl get all -n demo-k8s-basics` показывает работающее приложение.
2. **Доступность**:
   - приложение доступно внутри кластера по `app-svc` (через debug‑Pod);
   - приложение доступно с вашей машины (через port‑forward или NodePort/Ingress);
   - endpoint `/health` возвращает корректный ответ.
3. **Конфигурация**:
   - не менее 2 реплик в Deployment;
   - заданы `requests` и `limits` для ресурсов контейнера;
   - конфигурация вынесена в ConfigMap/Secret и не захардкожена в Pod.
4. **Rollout**:
   - вы один раз обновили образ на новую версию;
   - при необходимости выполнили `rollout undo` и понимаете, как это работает.

---

### 6. Stretch goals

1. **HorizontalPodAutoscaler (HPA)**  
   - настроить простой HPA по CPU для вашего Deployment;
   - нагрузить приложение и посмотреть, как растёт/падает количество Pod’ов.

2. **Примитивная observability**  
   - добавить простой endpoint для метрик и подключить Prometheus в том же кластере (`[[DevOps/08_Monitoring/prometheus]]`);
   - построить базовый дашборд в Grafana (`[[DevOps/08_Monitoring/grafana]]`).

3. **NetworkPolicy (для любопытных)**  
   - ограничить доступ к вашему Service только из определённых Pod’ов (`[[DevOps/04_Kubernetes/networking]]`).

---

Если вы уверенно прошли этот проект, вы действительно умеете выполнять базовый deploy приложения в Kubernetes и готовы к продвинутым объектам и Helm.

