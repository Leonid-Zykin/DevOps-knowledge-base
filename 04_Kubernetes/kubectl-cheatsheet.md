tags: [devops, kubernetes-kubectl]

## kubectl Cheatsheet — команды и примеры

> ⚡ Tip: Настройте алиас `alias k=kubectl` и `alias kgp='kubectl get pods'` для ускорения работы.

### Контексты и namespace

```bash
kubectl config get-contexts                         # список контекстов
kubectl config use-context prod-cluster             # переключиться на контекст
kubectl config current-context                      # текущий контекст

kubectl config set-context dev --namespace=dev      # namespace по умолчанию для контекста

kubectl get ns                                      # список namespace'ов
kubectl config set-context --current --namespace=dev
```

Временный namespace:

```bash
kubectl get pods -n kube-system
kubectl get svc -A                                  # во всех namespace'ах
```

### Базовые операции get/describe/delete

```bash
kubectl get pods                                   # pods в текущем ns
kubectl get pods -o wide                          # с IP и нодой
kubectl get pods -A                               # во всех ns

kubectl describe pod mypod                        # детали pod'а
kubectl describe deployment mydeploy
kubectl describe node worker-1

kubectl get svc                                   # сервисы
kubectl get ingress                               # ингрессы
kubectl get cm                                    # ConfigMap
kubectl get secret                                # секреты (названия)

kubectl delete pod mypod                          # удалить pod
kubectl delete pod mypod --grace-period=0 --force # жёсткое удаление
kubectl delete deployment mydeploy
kubectl delete ns temp-ns
```

Список ресурсов:

```bash
kubectl api-resources                              # все типы ресурсов
kubectl api-versions                               # версии API
```

### Создание и обновление ресурсов

```bash
kubectl apply -f deployment.yaml                   # создать/обновить
kubectl apply -f k8s/                              # рекурсивно по каталогу

kubectl delete -f deployment.yaml                  # удалить по манифесту

kubectl create deployment myapp --image=nginx:1.27
kubectl create configmap app-config --from-file=config.yml
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=secret
```

### Rollout и управление Deployment

```bash
kubectl rollout status deployment myapp
kubectl rollout history deployment myapp
kubectl rollout history deployment myapp --revision=2

kubectl rollout undo deployment myapp              # откат к предыдущей ревизии
kubectl rollout undo deployment myapp --to-revision=3
```

Scale:

```bash
kubectl scale deployment myapp --replicas=3
kubectl get deploy myapp -o wide
```

Update image:

```bash
kubectl set image deployment myapp \
  myapp=registry.example.com/myapp:1.2.3
```

### Logs, exec, port-forward

```bash
kubectl logs pod/mypod                             # логи
kubectl logs deploy/myapp                          # логи всех pod'ов deployment'а
kubectl logs -f mypod                              # tail -f
kubectl logs -f mypod -c sidecar                   # конкретный контейнер
kubectl logs mypod --since=10m                     # за последние 10 минут
kubectl logs mypod --previous                      # предыдущий контейнер (crash)

kubectl exec -it mypod -- bash                     # войти в pod
kubectl exec mypod -- ls /app
kubectl exec -it deploy/myapp -- bash              # в один из pod'ов deployment'а

kubectl port-forward pod/mypod 8080:80             # localhost:8080 → pod:80
kubectl port-forward svc/myservice 8080:80         # localhost:8080 → service:80
```

### JSONPath и кастомные колонки

```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

kubectl get pods \
  -o custom-columns='NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName'
```

Поиск по label:

```bash
kubectl get pods -l app=myapp
kubectl get svc -l app=myapp
kubectl get pods -l 'app in (myapp,worker)'
```

### Label и annotatе

```bash
kubectl label pod mypod env=dev
kubectl label pod mypod env-                      # удалить label env

kubectl annotate deployment myapp \
  deployer="ci-pipeline" build="1234"
```

### Dry-run и diff

```bash
kubectl apply -f deploy.yaml --dry-run=client -o yaml   # показать, что будет применено
kubectl diff -f deploy.yaml                             # diff текущего и манифеста
```

Используйте в CI перед `apply` (см. `[[05_CI-CD/gitlab-ci]]`, `[[05_CI-CD/github-actions]]`).

### Копирование файлов

```bash
kubectl cp myns/mypod:/app/logs ./logs
kubectl cp ./config.yml myns/mypod:/app/config.yml
```

### Describe событий и проблем

```bash
kubectl describe pod mypod                          # событие (Events)
kubectl get events --sort-by='.lastTimestamp'       # последние события
kubectl get events -A --field-selector type=Warning # только warning'и
```

### Node‑уровень

```bash
kubectl get nodes
kubectl describe node worker-1

kubectl cordon worker-1                             # пометить как unschedulable
kubectl uncordon worker-1

kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data
```

### Kubeconfig и контексты (JSONPath)

```bash
kubectl config view
kubectl config view -o jsonpath='{.contexts[*].name}'
```

### Быстрые one-liners

```bash
# Все поды, не в Running
kubectl get pods -A --field-selector=status.phase!=Running

# Подробный список подов с node и IP
kubectl get pods -A -o custom-columns=NS:.metadata.namespace,NAME:.metadata.name,NODE:.spec.nodeName,IP:.status.podIP,STATUS:.status.phase

# Все контейнеры с рестартами > 0
kubectl get pods -A \
  -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}' | awk '$3 > 0'
```

Связанные заметки: `[[core-concepts]]`, `[[workloads]]`, `[[networking]]`, `[[storage]]`, `[[rbac-and-security]]`, `[[helm]]`, `[[troubleshooting]]`.

## Gotchas

- **Контекст/namespace не тот**  
  - **Проблема**: применяете манифесты или удаляете ресурсы не в том кластере/namespace.  
  - **Решение**: всегда проверять `kubectl config current-context` и `--namespace`; использовать явный `-n` в продовых командах.

- **`kubectl delete pod` для управления rollout'ом**  
  - **Проблема**: ручное удаление pod'ов вместо управления Deployment/StatefulSet.  
  - **Решение**: использовать `kubectl rollout`, `kubectl set image`, масштабирование Deployment/StatefulSet (см. `[[workloads]]`).

- **Отсутствие `--dry-run`/`diff` в CI**  
  - **Проблема**: неожиданные изменения ресурсов при `apply`.  
  - **Решение**: в пайплайнах сначала `kubectl diff`/`--dry-run=server`, потом `apply`.

> ⚠️ Warning: Любые массовые операции вида `kubectl delete po --all -A` или `kubectl delete ns` без точного понимания последствий могут привести к тотальному простою кластера. В проде применяйте только таргетированные команды и обязательно проверяйте селекторы (`-l`, `-A`, `-n`).

