## Module 06 — Kubernetes Basics Practice

Практика рассчитана на локальный кластер (**minikube** или **kind**). Перед началом убедитесь, что:

- `kubectl` настроен и `kubectl get nodes` показывает хотя бы одну `Ready`‑ноду;
- есть доступ в интернет для загрузки образов.

Опирайтесь на:

- `[[DevOps/04_Kubernetes/core-concepts]]`
- `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`
- `[[DevOps/04_Kubernetes/troubleshooting]]`

---

### Task 1 — Проверка кластера и контекста

- **Goal**: Убедиться, что вы работаете в правильном кластере/namespace.
- **Steps**:
  1. Выполните:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl get nodes -o wide
kubectl get ns
```

  2. Установите namespace по умолчанию (например, `dev`), создав его при необходимости.
- **Expected result**: Вы знаете, в каком контексте и namespace будете работать.

---

### Task 2 — Ваш первый Pod

- **Goal**: Создать простой Pod и посмотреть его жизненный цикл.
- **Steps**:
  1. Напишите манифест `nginx-pod.yaml` с Pod’ом из теории (nginx:1.27).
  2. Примените его:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
```

  3. Зайдите внутрь:

```bash
kubectl exec -it nginx-pod -- bash
```

- **Expected result**: Вы видите Pod в статусе `Running` и можете выполнить команды внутри него.

---

### Task 3 — Deployment + масштабирование

- **Goal**: Управлять репликами через Deployment.
- **Steps**:
  1. Опишите Deployment `nginx-deploy` на 2 реплики.
  2. Примените:

```bash
kubectl apply -f nginx-deploy.yaml
kubectl get deploy
kubectl get rs
kubectl get pods -l app=nginx
```

  3. Масштабируйте до 5 реплик:

```bash
kubectl scale deploy nginx-deploy --replicas=5
kubectl get pods -l app=nginx
```

- **Expected result**: Количество Pod’ов плавно меняется, Deployment в статусе `Available`.

---

### Task 4 — Service ClusterIP и доступ из debug‑под’а

- **Goal**: Настроить доступ к приложению через Service внутри кластера.
- **Steps**:
  1. Создайте Service `nginx-svc` типа `ClusterIP`, селектор `app=nginx`, порт 80.
  2. Создайте debug‑Pod (busybox/alpine):

```bash
kubectl run debug --image=busybox:1.36 -it --restart=Never -- sh
```

  3. Из debug‑под’а:

```sh
nslookup nginx-svc
curl -v http://nginx-svc
```

- **Expected result**: Внутри кластера сервис доступен по имени `nginx-svc`.

---

### Task 5 — NodePort и доступ снаружи (minikube/kind)

- **Goal**: Открыть сервис снаружи кластера.
- **Steps**:
  1. Измените Service на тип `NodePort` или создайте отдельный `nginx-nodeport`.
  2. Посмотрите:

```bash
kubectl get svc
```

  3. Узнайте IP ноды (или используйте `minikube ip`) и выполните:

```bash
curl -v http://<node_ip>:<nodePort>
```

- **Expected result**: Доступ к nginx из вашей машины (хоста).

---

### Task 6 — ConfigMap + envFrom

- **Goal**: Вынести конфигурацию в ConfigMap и использовать её в Deployment.
- **Steps**:
  1. Создайте ConfigMap `web-config` с:

```yaml
APP_ENV: "dev"
WELCOME_MESSAGE: "Hello from ConfigMap"
```

  2. Модифицируйте Deployment так, чтобы контейнер имел эти переменные окружения через `envFrom.configMapRef`.
  3. Внутри Pod’а проверьте:

```bash
kubectl exec -it <pod> -- env | grep WELCOME
```

- **Expected result**: Переменные доступны в контейнере и могут использоваться приложением.

---

### Task 7 — Secret для пароля БД

- **Goal**: Хранить чувствительные данные в Secret.
- **Steps**:
  1. Создайте Secret `db-secret` с ключом `DB_PASSWORD=secret`.
  2. Модифицируйте любой Deployment (например, простого API или nginx) так, чтобы:
     - переменная окружения `DB_PASSWORD` приходила из Secret.
  3. Проверьте внутри Pod’а, что переменная установлена.
- **Expected result**: Secret корректно монтируется как env, не появляется в манифесте в открытом виде.

---

### Task 8 — Обновление образа и rollback

- **Goal**: Отработать обновление и откат Deployment.
- **Steps**:
  1. Обновите образ Deployment:

```bash
kubectl set image deploy/nginx-deploy nginx=nginx:1.27.1
kubectl rollout status deploy/nginx-deploy
```

  2. Посмотрите историю:

```bash
kubectl rollout history deploy/nginx-deploy
```

  3. Выполните откат:

```bash
kubectl rollout undo deploy/nginx-deploy
```

- **Expected result**: Вы видите, как Deployment обновляется и откатывается.

---

### Task 9 — «Break it & fix it»: CrashLoopBackOff

- **Goal**: Потренироваться диагностировать падающий Pod.
- **Steps**:
  1. Создайте Deployment, в котором контейнер выполняет команду `exit 1` или обращается к несуществующему бинарю.
  2. Посмотрите:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

  3. Исправьте манифест и сделайте `kubectl apply -f ...`.
- **Expected result**: Вы умеете читать причины CrashLoopBackOff и устранять их.

---

### Task 10 — «Break it & fix it»: Service без endpoints

- **Goal**: Понять, что такое Service без endpoints.
- **Steps**:
  1. Создайте Service, селектор которого не совпадает с label’ами Pod’ов.
  2. Посмотрите:

```bash
kubectl get endpoints <svc-name>
```

  3. Попробуйте обратиться к Service из debug‑Pod’а (curl/HTTP).
  4. Почините селектор так, чтобы endpoints появились.
- **Expected result**: Вы понимаете связь селекторов и endpoints.

---

### Task 11 — Port‑forward для локальной отладки

- **Goal**: Подключиться к сервису без настройки NodePort/Ingress.
- **Steps**:
  1. Выберите любой Pod или Service.
  2. Выполните:

```bash
kubectl port-forward svc/nginx-svc 8080:80
```

  3. На хосте:

```bash
curl http://127.0.0.1:8080
```

- **Expected result**: Вы умеете использовать port‑forward для локальной отладки.

---

### Task 12 — Собственный namespace для приложения

- **Goal**: Логически изолировать своё приложение.
- **Steps**:
  1. Создайте namespace `demo` и переключитесь в него (через контекст или `-n demo`).
  2. Перенесите все объекты (Deployment, Service, ConfigMap, Secret) в этот namespace.
  3. Убедитесь, что в `default` namespace ничего лишнего не осталось.
- **Expected result**: Приложение живёт в отдельном namespace, и вы умеете с ним работать.

---

## Итог

После выполнения заданий вы должны:

- уверенно создавать и обновлять Deployment’ы;
- настраивать Service и проверять связность через endpoints и debug‑Pod;
- выносить конфигурацию в ConfigMap/Secret;
- понимать базовый troubleshooting CrashLoopBackOff и сервисов без endpoints.

Эти навыки — основа для продвинутых объектов (StatefulSet, DaemonSet, Jobs) и Helm в следующих модулях.

