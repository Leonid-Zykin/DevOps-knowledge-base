tags: [devops, kubernetes-troubleshooting]

## Kubernetes Troubleshooting — CrashLoopBackOff, Pending, OOMKilled

> ⚡ Tip: Базовый порядок: `kubectl get` → `kubectl describe` → `kubectl logs` → `kubectl exec` → `events` → node‑уровень.

### CrashLoopBackOff

```bash
kubectl get pods
kubectl describe pod mypod
kubectl logs mypod
kubectl logs mypod --previous    # предыдущий контейнер (до рестарта)
```

Типичные причины:

- приложение падает при старте (ошибка конфигурации, миграции, зависимостей);
- неправильная команда `command`/`args`;
- зависимые сервисы недоступны (БД, очередь).

Действия:

```bash
kubectl exec -it mypod -- /bin/sh
kubectl describe deploy mydeploy
kubectl get cm,secret
```

### Pod в состоянии Pending

```bash
kubectl get pods
kubectl describe pod mypod
```

Причины:

- нет подходящих нод (resources, taints/tolerations, nodeSelector/nodeAffinity);
- PVC в состоянии `Pending`;
- проблемы с CNI/сетью.

Проверки:

```bash
kubectl get nodes
kubectl describe nodes | grep -A5 Taints
kubectl get pvc
kubectl describe pvc <name>
kubectl get events --sort-by='.lastTimestamp'
```

### OOMKilled / ResourceLimits

```bash
kubectl get pod mypod -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
kubectl describe pod mypod | grep -i -A5 "State"
```

Если `reason: OOMKilled`:

- увеличить `resources.limits.memory`;
- оптимизировать потребление;
- добавить наблюдаемость (см. `[[08_Monitoring/prometheus]]`).

### ImagePullBackOff / ErrImagePull

```bash
kubectl describe pod mypod | grep -A3 "Containers:"
```

Проверки:

- корректность образа (`image: repo/name:tag`);
- доступ к реестру (Secret с docker credentials);
- права ServiceAccount.

### Не работает доступ через Service/Ingress

Порядок:

1. Pod‑ы:
   ```bash
   kubectl get pods -l app=api -o wide
   kubectl logs deploy/api
   ```
2. Service:
   ```bash
   kubectl get svc api-svc
   kubectl get endpoints api-svc
   ```
3. Ingress:
   ```bash
   kubectl get ingress
   kubectl describe ingress api-ingress
   kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
   ```

Смотрите `[[networking]]` и `[[11_Networking/troubleshooting]]`.

### Node‑уровень: NotReady, DiskPressure

```bash
kubectl get nodes
kubectl describe node worker-1
```

Причины:

- `DiskPressure`, `MemoryPressure`, `PIDPressure`;
- kubelet не может запуститься (systemd, cgroups).

Дальнейшие шаги:

- смотреть логи kubelet (`journalctl -u kubelet`);
- проверять диск, inode, RAM (см. `[[01_Linux/linux-cheatsheet]]`, `[[01_Linux/processes-and-services]]`).

### Events

```bash
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -A --field-selector type=Warning
```

### Debug pod

```bash
kubectl run -it debug --image=busybox:1.36 --restart=Never -- sh
kubectl delete pod debug
```

Или ephemeral containers (если доступны):

```bash
kubectl debug pod/mypod -it --image=busybox:1.36
```

Связанные заметки: `[[kubectl-cheatsheet]]`, `[[core-concepts]]`, `[[workloads]]`, `[[networking]]`, `[[storage]]`.

## Gotchas

- **Фокус только на Pod, игнорируя Deployment/ReplicaSet**  
  - **Проблема**: лечение симптома (удаление Pod), а не причины (конфиг деплоймента).  
  - **Решение**: анализировать Deployment/StatefulSet/DaemonSet (`kubectl describe deploy/...`), не править Pod напрямую.

- **Игнорирование событий (Events)**  
  - **Проблема**: ошибки планировщика/NetworkPolicy/Storage видны только в events.  
  - **Решение**: всегда смотреть `kubectl get events --sort-by='.lastTimestamp'` при непонятном поведении.

- **Ручной доступ и правки в контейнерах вместо Git/Helm**  
  - **Проблема**: конфигурация дрейфует, изменения не воспроизводимы.  
  - **Решение**: все изменения оформлять через манифесты/Helm/Argo CD (`[[helm]]`, `[[05_CI-CD/argocd]]`).

> ⚠️ Warning: Массовое использование `kubectl delete pod --all -A` или `kubectl delete node` как «способа починки» кластера может привести к cascade‑падению критичных сервисов и долговременным outage. Troubleshooting должен быть точечным, с понятной гипотезой и обратным планом.

