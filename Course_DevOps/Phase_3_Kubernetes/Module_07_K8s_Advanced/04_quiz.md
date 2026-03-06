## Module 07 — Kubernetes Advanced Quiz

Ориентируйтесь на:
- `[[DevOps/04_Kubernetes/workloads]]`
- `[[DevOps/04_Kubernetes/storage]]`
- `[[DevOps/04_Kubernetes/rbac-and-security]]`
- `[[DevOps/04_Kubernetes/networking]]`

---

### Вопрос 1 — StatefulSet

Чем StatefulSet принципиально отличается от Deployment для stateful‑приложений?

a) Ничем, это синонимы  
b) StatefulSet даёт стабильные имена Pod’ов и порядок создания/удаления, часто используется с PVC  
c) StatefulSet нельзя масштабировать  
d) StatefulSet работает только с БД из облака

---

### Вопрос 2 — Headless Service

Зачем для StatefulSet часто создают Service с `clusterIP: None`?

a) Чтобы сэкономить IP  
b) Чтобы получить стабильные DNS‑имена для каждого Pod’а (например, `db-0.db.ns.svc.cluster.local`)  
c) Чтобы отключить балансировку  
d) Это устаревшая практика

---

### Вопрос 3 — DaemonSet

Что гарантирует DaemonSet?

a) Ровно один Pod на весь кластер  
b) По одному Pod’у на каждой (подходящей) ноде кластера  
c) Минимум два Pod’а на ноду  
d) Только запуск на master‑нодах

---

### Вопрос 4 — Job vs CronJob

В чём разница между Job и CronJob?

a) Job — один раз, CronJob — по расписанию создаёт Job’ы  
b) Job — для БД, CronJob — для веб‑сервисов  
c) Разницы нет  
d) CronJob устарел

---

### Вопрос 5 — PVC и StorageClass

Что такое PersistentVolumeClaim (PVC)?

a) Тип тома внутри Pod’а  
b) Запрос на выделение хранилища; кластер (или StorageClass) выделяет PV и связывает с PVC  
c) Резервная копия диска  
d) Сетевое хранилище только для StatefulSet

---

### Вопрос 6 — AccessModes

Какой режим доступа к тому подходит для «один Pod на одной ноде читает/пишет»?

a) ReadOnlyMany  
b) ReadWriteMany  
c) ReadWriteOnce  
d) ReadWriteNone

---

### Вопрос 7 — RBAC

Что связывает ServiceAccount с набором прав в RBAC?

a) Только ClusterRole  
b) RoleBinding или ClusterRoleBinding связывает Role/ClusterRole с Subject (в т.ч. ServiceAccount)  
c) Secret  
d) ConfigMap

---

### Вопрос 8 — OOMKilled

Pod перезапускается, в describe видно `lastState.terminated.reason: OOMKilled`. Что это значит?

a) Контейнер превысил лимит памяти (memory limit) и был убит ядром  
b) Нехватка CPU  
c) Ошибка приложения  
d) Сетевой таймаут

---

### Вопрос 9 — NetworkPolicy

Что делает NetworkPolicy в Kubernetes?

a) Настраивает DNS  
b) Ограничивает входящий/исходящий трафик между Pod’ами (L3/L4) по селекторам  
c) Управляет Ingress Controller  
d) Шифрует трафик

---

### Вопрос 10 — PodDisruptionBudget

Зачем нужен PodDisruptionBudget (PDB)?

a) Для ограничения количества Pod’ов в namespace  
b) Чтобы при добровольном выводе нод (drain) или автоскейлинге не выбить сразу все реплики приложения  
c) Для резервного копирования  
d) Для ускорения деплоя

---

> [!note]- Answers
> **1** — b) StatefulSet даёт стабильные имена и порядок, часто с PVC. `[[DevOps/04_Kubernetes/workloads]]`.  
> **2** — b) Headless Service даёт стабильные DNS‑имена для каждого Pod’а.  
> **3** — b) По одному Pod’у на каждой подходящей ноде.  
> **4** — a) Job — одноразовая задача, CronJob по расписанию создаёт Job’ы.  
> **5** — b) PVC — запрос на хранилище; провайдер/StorageClass выделяет PV. `[[DevOps/04_Kubernetes/storage]]`.  
> **6** — c) ReadWriteOnce (RWO).  
> **7** — b) RoleBinding/ClusterRoleBinding связывает Role/ClusterRole с Subject. `[[DevOps/04_Kubernetes/rbac-and-security]]`.  
> **8** — a) Превышен memory limit, контейнер убит. `[[DevOps/04_Kubernetes/troubleshooting]]`.  
> **9** — b) Ограничивает трафик между Pod’ами. `[[DevOps/04_Kubernetes/networking]]`.  
> **10** — b) Защита от одновременного выбивания слишком многих реплик. `[[DevOps/04_Kubernetes/workloads]]`.
