## Module 06 — Kubernetes Basics Quiz

Ориентируйтесь на:

- `[[DevOps/04_Kubernetes/core-concepts]]`
- `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`
- `[[DevOps/04_Kubernetes/troubleshooting]]`

---

### Вопрос 1 — Pod vs Deployment

Какое утверждение наиболее точное?

a) Pod и Deployment — одно и то же  
b) Pod — минимальная единица деплоя, Deployment управляет набором Pod’ов (через ReplicaSet)  
c) Deployment используется только для БД, Pod — для статического контента  
d) Deployment — это контейнер, Pod — это образ

---

### Вопрос 2 — Selector и labels

Что произойдёт, если `spec.selector.matchLabels` Deployment’а **не совпадает** с `metadata.labels` шаблона Pod’а (`spec.template.metadata.labels`)?

a) Ничего страшного, Kubernetes сам подберёт правильные Pod’ы  
b) Deployment не будет управлять созданными Pod’ами и покажет 0 реплик  
c) Pod’ы будут созданы, но Service не сможет к ним подключиться  
d) Кластер упадёт

---

### Вопрос 3 — Service и endpoints

Какой командой лучше всего проверить, какие Pod’ы «подвешены» под Service?

a) `kubectl get pods`  
b) `kubectl get endpoints <svc-name>`  
c) `kubectl get nodes`  
d) `kubectl describe node`

---

### Вопрос 4 — Типы Service

Какой тип Service обеспечивает доступ **только внутри кластера**?

a) NodePort  
b) LoadBalancer  
c) ClusterIP  
d) ExternalName

---

### Вопрос 5 — ConfigMap vs Secret

Чем принципиально отличается Secret от ConfigMap?

a) Secret шифруется всегда, а ConfigMap — нет  
b) Secret предназначен для чувствительных данных (пароли, ключи), ConfigMap — для обычной конфигурации; Secret хранится в base64 и может быть дополнительно зашифрован (`encryption at rest`)  
c) ConfigMap хранится в другом etcd‑кластере  
d) Разницы нет, используются взаимозаменяемо

---

### Вопрос 6 — kubectl logs/exec

Какой набор команд вы будете использовать для базовой диагностики Pod’а `myapp-12345`?

a) `kubectl get nodes`, `kubectl delete pod myapp-12345`  
b) `kubectl logs myapp-12345`, `kubectl exec -it myapp-12345 -- sh`  
c) `kubectl rollout undo deploy/myapp`  
d) `kubectl describe node`

---

### Вопрос 7 — CrashLoopBackOff

Вы видите Pod в состоянии `CrashLoopBackOff`. Какие шаги диагностики наиболее уместны?

a) Сразу удалить Pod и забыть  
b) `kubectl describe pod`, затем `kubectl logs pod --previous`  
c) `kubectl get nodes`, затем перезапустить кластер  
d) Ничего, это нормальное состояние

---

### Вопрос 8 — Namespace

Зачем использовать отдельные Namespaces?

a) Только для экономии памяти  
b) Для логического разделения окружений/команд и настройки квот/прав доступа  
c) Для ускорения kubectl  
d) Никакой пользы нет

---

### Вопрос 9 — Ingress

Что делает ресурс Ingress в Kubernetes?

a) Настраивает внутреннюю сеть Pod’ов  
b) Описывает правила маршрутизации HTTP/HTTPS‑трафика к Service’ам через Ingress Controller  
c) Управляет volume’ами и storage  
d) Настраивает RBAC

---

### Вопрос 10 — Kubectl context

Почему важно проверять `kubectl config current-context` перед выполнением разрушительных операций?

a) Команда бесполезна, можно её игнорировать  
b) Чтобы убедиться, что вы работаете в правильном кластере/namespace и не удалите ресурсы в проде по ошибке  
c) Чтобы ускорить apply  
d) Чтобы включить отладку

---

> [!note]- Answers
> **Вопрос 1**  
> b) Pod — минимальная единица деплоя, Deployment управляет наборами Pod’ов. См. `[[DevOps/04_Kubernetes/core-concepts]]`.  
>
> **Вопрос 2**  
> b) Deployment не будет видеть свои Pod’ы и покажет 0 управляемых реплик; это типичная ошибка.  
>
> **Вопрос 3**  
> b) `kubectl get endpoints <svc-name>`.  
> Показывает IP/порты Pod’ов, на которые Service отправляет трафик.  
>
> **Вопрос 4**  
> c) ClusterIP.  
> Это дефолтный тип Service для внутреннего доступа.  
>
> **Вопрос 5**  
> b) Secret предназначен для чувствительных данных, хранится в base64 и может быть зашифрован; ConfigMap — для обычной конфигурации. См. `[[DevOps/04_Kubernetes/core-concepts]]` и `[[DevOps/10_Security/secrets-management]]`.  
>
> **Вопрос 6**  
> b) `kubectl logs` и `kubectl exec`.  
>
> **Вопрос 7**  
> b) `kubectl describe pod` + `kubectl logs --previous`.  
> Это описано в `[[DevOps/04_Kubernetes/troubleshooting]]`.  
>
> **Вопрос 8**  
> b) Для разделения окружений и управления правами/квотами.  
>
> **Вопрос 9**  
> b) Ingress управляет HTTP/HTTPS‑маршрутизацией к Service’ам через Ingress Controller. См. `[[DevOps/04_Kubernetes/networking]]`.  
>
> **Вопрос 10**  
> b) Чтобы не выполнить опасные команды против неправильного кластера/namespace. См. `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`.  

