## Module 07 — Kubernetes Advanced Practice

Кластер (minikube/kind) и `kubectl` должны быть настроены. Опирайтесь на `[[DevOps/04_Kubernetes/workloads]]`, `[[DevOps/04_Kubernetes/storage]]`, `[[DevOps/04_Kubernetes/rbac-and-security]]`, `[[DevOps/04_Kubernetes/networking]]`.

---

### Task 1 — StatefulSet с volumeClaimTemplates

- **Goal**: Запустить stateful‑приложение с персистентным томом.
- **Steps**:
  1. Создайте Headless Service (`clusterIP: None`) для StatefulSet.
  2. Создайте StatefulSet (например, Redis или простой app с одним репликой), использующий `volumeClaimTemplates` с `storageClassName` (стандартный в кластере или `standard`).
  3. Примените манифесты, дождитесь `Running`.
  4. Проверьте имена Pod’ов (`redis-0`, …) и наличие PVC (`data-redis-0`).
- **Expected result**: Pod’ы имеют стабильные имена, данные сохраняются при пересоздании Pod’а.

---

### Task 2 — Доступ к StatefulSet по DNS

- **Goal**: Убедиться, что Pod’ы StatefulSet резолвятся по имени.
- **Steps**:
  1. Из debug‑Pod’а выполните `nslookup redis-0.redis` (или имя вашего headless service).
  2. Подключитесь к приложению по имени (например, `redis-0.redis:6379`).
- **Expected result**: Понимание стабильных DNS‑имён для stateful‑под’ов.

---

### Task 3 — DaemonSet (node‑exporter или аналог)

- **Goal**: Запустить агент на каждой ноде.
- **Steps**:
  1. Создайте DaemonSet с образом `prom/node-exporter:v1.8.2`, порт 9100.
  2. Примените, проверьте: на каждой ноде должен быть ровно один Pod.
  3. Создайте Service для доступа к метрикам (ClusterIP или NodePort).
- **Expected result**: DaemonSet развёрнут, метрики доступны (curl с любой ноды или через port‑forward).

---

### Task 4 — Job для «миграции»

- **Goal**: Выполнить одноразовую задачу через Job.
- **Steps**:
  1. Создайте Job с простой командой (например, `echo "migration done"` и sleep 5).
  2. Дождитесь завершения (`kubectl get jobs`, `kubectl logs job/<name>`).
  3. Проверьте, что Job в статусе `Complete` и не перезапускается.
- **Expected result**: Job выполняется один раз и завершается.

---

### Task 5 — CronJob

- **Goal**: Настроить периодическую задачу.
- **Steps**:
  1. Создайте CronJob с расписанием `*/2 * * * *` (каждые 2 минуты) и простой командой.
  2. Дождитесь появления Job’ов (`kubectl get jobs`, `kubectl get cronjobs`).
  3. Установите `successfulJobsHistoryLimit: 2`, `failedJobsHistoryLimit: 2`.
- **Expected result**: CronJob создаёт Job’ы по расписанию; история ограничена.

---

### Task 6 — PVC и подключение к Pod’у

- **Goal**: Выделить том через PVC и смонтировать в Pod.
- **Steps**:
  1. Создайте PVC на 1Gi, `ReadWriteOnce`, без указания storageClass (или с дефолтным).
  2. Создайте Deployment или Pod с volume `persistentVolumeClaim` и смонтируйте в `/data`.
  3. Запишите файл в `/data`, удалите Pod, создайте новый с тем же PVC — файл должен сохраниться.
- **Expected result**: Данные переживают пересоздание Pod’а.

---

### Task 7 — Resource limits и OOMKilled (опционально)

- **Goal**: Увидеть поведение при нехватке памяти.
- **Steps**:
  1. Создайте Deployment с контейнером, который быстро потребляет память (например, тестовый образ или `stress`), и установите маленький `limits.memory` (например, 64Mi).
  2. Дождитесь рестартов и проверьте: `kubectl describe pod` → `lastState.terminated.reason: OOMKilled`.
  3. Увеличьте limit и убедитесь, что Pod стабилен.
- **Expected result**: Понимание связи limits и OOMKilled. См. `[[DevOps/04_Kubernetes/troubleshooting]]`.

---

### Task 8 — ServiceAccount и RoleBinding

- **Goal**: Выдать приложению ограниченные права.
- **Steps**:
  1. Создайте ServiceAccount `app-sa`.
  2. Создайте Role, разрешающий только `get`, `list` для `pods` в namespace.
  3. Создайте RoleBinding, привязавший Role к `app-sa`.
  4. Запустите Pod с `serviceAccountName: app-sa` и изнутри (например, с установленным `kubectl`) проверьте: `kubectl get pods` — должно работать; `kubectl delete pod ...` — при отсутствии права delete — нет.
- **Expected result**: Понимание минимальных прав через RBAC. См. `[[DevOps/04_Kubernetes/rbac-and-security]]`.

---

### Task 9 — SecurityContext (readOnlyRootFilesystem)

- **Goal**: Запустить контейнер с read‑only корневой ФС.
- **Steps**:
  1. Создайте Deployment с `securityContext.readOnlyRootFilesystem: true` и `volumeMount` для `/tmp` (emptyDir).
  2. Убедитесь, что приложение не пишет в корень и может писать в `/tmp`.
  3. При необходимости задайте `runAsNonRoot: true` и `runAsUser`.
- **Expected result**: Pod работает с ограничениями security context.

---

### Task 10 — PodDisruptionBudget

- **Goal**: Защитить Deployment от одновременного выбивания всех реплик.
- **Steps**:
  1. Разверните Deployment с 3 репликами.
  2. Создайте PDB с `minAvailable: 2` для этого Deployment.
  3. Попробуйте выполнить `kubectl drain` на ноде (если однокластерная среда — симулятивно или на многонодовом кластере) и понаблюдайте за поведением.
- **Expected result**: Понимание того, что PDB ограничивает одновременное удаление Pod’ов.

---

## Итог

После практики вы умеете:
- использовать StatefulSet, DaemonSet, Job, CronJob;
- работать с PVC и подключать тома к Pod’ам;
- задавать ресурсы и понимать OOMKilled;
- настраивать RBAC (SA, Role, RoleBinding) и SecurityContext;
- применять PDB там, где нужна устойчивость к drain’у.

Эти навыки нужны для мини‑проекта (stateful app) и для перехода к Helm и продакшен‑сценариям.
