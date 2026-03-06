## Module 07 — Mini‑project: Stateful App (PostgreSQL + App)

### 1. Цель проекта

**Задача**: развернуть в Kubernetes stateful‑приложение:

- **PostgreSQL** (StatefulSet + headless Service + PVC);
- **ваше приложение** (Deployment), подключающееся к БД;
- ConfigMap/Secret для конфигурации и паролей;
- при необходимости — Service и доступ снаружи (NodePort/port‑forward).

Оценочное время: **5–10 часов**.

---

### 2. Архитектура

```text
[App Deployment]  →  [PostgreSQL StatefulSet]
       |                        |
   app-svc (ClusterIP)    postgres (headless)
       |                        |
   PVC: app-data           volumeClaimTemplate: data
```

- App — stateless, масштабируется репликами.
- PostgreSQL — один инстанс (или три при желании), данные в PVC.

---

### 3. Требования

1. **Namespace**: отдельный (например, `stateful-demo`).
2. **PostgreSQL**:
   - StatefulSet с `replicas: 1` (или 3 для практики);
   - Headless Service `postgres`;
   - `volumeClaimTemplates` для `/var/lib/postgresql/data`;
   - пароль/пользователь из Secret.
3. **App**:
   - Deployment с 2 репликами;
   - переменная подключения к БД из Secret (например, `DATABASE_URL` или `POSTGRES_HOST`, `POSTGRES_PASSWORD`);
   - `depends_on` через init‑контейнер или просто порядок деплоя; опционально readiness probe на проверку БД.
4. **Сеть**: App подключается к `postgres-0.postgres` (или к Service `postgres` на 5432).
5. **Документация**: как запустить, как проверить подключение, как сделать бэкап/восстановление PVC (кратко).

---

### 4. Пошаговый план

#### Шаг 1 — Secret и ConfigMap

- Secret: `postgres-password`, `app-db-url` (или отдельные ключи).
- ConfigMap: `APP_ENV`, `LOG_LEVEL` и т.п.

#### Шаг 2 — Headless Service для PostgreSQL

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

#### Шаг 3 — StatefulSet для PostgreSQL

- image: `postgres:16`;
- env из Secret (POSTGRES_PASSWORD, POSTGRES_USER, POSTGRES_DB);
- volumeClaimTemplates с accessMode ReadWriteOnce, размер например 5Gi;
- при необходимости readiness probe: `pg_isready`.

#### Шаг 4 — Deployment приложения

- image — ваш образ из Phase 2;
- envFrom: Secret и ConfigMap;
- переменная с хостом БД: `postgres-0.postgres` или `postgres`.

#### Шаг 5 — Service для приложения

- ClusterIP (или NodePort) для доступа к app;
- проверка через `kubectl port-forward` или curl из debug‑Pod’а.

#### Шаг 6 — Проверка и отказоустойчивость

- Перезапуск Pod’а PostgreSQL: данные должны сохраниться.
- Масштабирование app: все реплики должны подключаться к одной БД.

---

### 5. Acceptance criteria

- [ ] StatefulSet PostgreSQL запущен, данные хранятся в PVC.
- [ ] App (Deployment) подключается к PostgreSQL и успешно выполняет запросы (например, выборка из таблицы).
- [ ] Secret и ConfigMap используются, пароли не в манифестах в открытом виде.
- [ ] После удаления Pod’а PostgreSQL и его пересоздания данные остаются.
- [ ] В README описаны команды запуска, проверки и (кратко) работа с PVC.

---

### 6. Stretch goals

- Реплики PostgreSQL (3 инстанса) + описание ограничений (синхронная репликация не настроена — только как практика StatefulSet).
- Настройка ResourceQuota/LimitRange в namespace.
- Простая NetworkPolicy: доступ к PostgreSQL только от Pod’ов с label `app=myapp`.

---

См. `[[DevOps/04_Kubernetes/workloads]]`, `[[DevOps/04_Kubernetes/storage]]`, `[[DevOps/04_Kubernetes/rbac-and-security]]`.
