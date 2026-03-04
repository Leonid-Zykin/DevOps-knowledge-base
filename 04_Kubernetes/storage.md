tags: [devops, kubernetes-storage]

## Kubernetes Storage — PV, PVC, StorageClass

> ⚡ Tip: В большинстве managed‑кластеров динамическое выделение дисков работает через `StorageClass`: достаточно создать PVC c нужным классом и размером.

### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

Команды:

```bash
kubectl get storageclass
kubectl describe storageclass fast-ssd
```

### PersistentVolumeClaim (PVC)

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

Использование в Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: myapp:latest
      volumeMounts:
        - name: data
          mountPath: /var/lib/app
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: app-data
```

### PersistentVolume (PV) — статический

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 10.0.0.10
    path: /exports/data
```

Команды:

```bash
kubectl get pv
kubectl get pvc
kubectl describe pv pv-nfs
kubectl describe pvc app-data
```

### AccessModes

| Режим         | Описание                           | Типичный сценарий                  |
|--------------|------------------------------------|------------------------------------|
| ReadWriteOnce (RWO) | монтирование на один node (чт/запись) | обычный диск для одной поды     |
| ReadOnlyMany (ROX)  | many nodes, только чтение     | общие данные                        |
| ReadWriteMany (RWX) | many nodes, чт/запись         | общая файловая система (NFS/CSI)   |

### Резайз томов

Условие: `allowVolumeExpansion: true` в StorageClass.

```bash
kubectl edit pvc app-data
# увеличить spec.resources.requests.storage: 20Gi → 50Gi
```

### StatefulSet и PVC

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: "redis"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7
          volumeMounts:
            - name: data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 10Gi
```

Имена PVC: `data-redis-0`, `data-redis-1`, ...

### Ephemeral storage (emptyDir)

```yaml
volumes:
  - name: cache
    emptyDir:
      sizeLimit: "1Gi"

containers:
  - name: app
    volumeMounts:
      - name: cache
        mountPath: /tmp/cache
```

- данные теряются при перезапуске Pod'а;
- удобно для кэшей, временных файлов.

Связанные: `[[workloads]]`, `[[core-concepts]]`, `[[08_Monitoring/elk-stack]]` (лог‑хранилища), `[[09_Cloud/aws-cheatsheet]]` (EBS/EFS).

## Gotchas

- **PVC без StorageClass в кластере с обязательным SC**  
  - **Проблема**: PVC застревает в состоянии `Pending`.  
  - **Решение**: указать `storageClassName` или настроить default StorageClass.

- **ReclaimPolicy=Delete на важных PV**  
  - **Проблема**: при удалении PVC underlying диск тоже удаляется (данные теряются).  
  - **Решение**: для критичных данных использовать `Retain` и управлять очисткой вручную.

- **Попытка расшарить RWO‑том между несколькими Pod на разных нодах**  
  - **Проблема**: PV не монтируется, Pod'ы в `ContainerCreating`.  
  - **Решение**: либо использовать RWX‑способ хранилища (NFS, EFS, CephFS), либо ограничить Pod'ы одной нодой (StatefulSet) или разделить данные.

> ⚠️ Warning: Любые операции удаления PVC/PV в прод‑кластере без понимания `ReclaimPolicy` и привязанных дисков в облаке могут привести к безвозвратной потере БД и файлов. Перед удалением всегда проверяйте `kubectl describe pv/pvc` и состояние дисков в облаке.

