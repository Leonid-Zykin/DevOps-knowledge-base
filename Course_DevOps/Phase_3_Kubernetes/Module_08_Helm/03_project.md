## Module 08 — Mini‑project: Package Your App as a Helm Chart

### 1. Цель проекта

**Задача**: упаковать ваше Kubernetes‑приложение (из Module_06/07) в Helm‑чарт:

- параметризовать образ, реплики, ресурсы, env;
- поддержать разные окружения через values‑файлы;
- обеспечить установку, upgrade и rollback.

Оценочное время: **4–8 часов**.

---

### 2. Требования к чарту

1. **Структура**:
   - Chart.yaml с корректными version и appVersion;
   - values.yaml с дефолтами;
   - templates: Deployment, Service, ConfigMap, Secret (опционально Ingress).
2. **Параметризация**:
   - image.repository, image.tag, image.pullPolicy;
   - replicaCount;
   - resources (requests/limits);
   - env (через values);
   - service.type (ClusterIP/NodePort).
3. **Окружения**:
   - values-dev.yaml, values-prod.yaml (или values-staging.yaml);
   - различие в репликах, ресурсах, env.
4. **Документация**:
   - README с описанием параметров и примерами установки.

---

### 3. Пошаговый план

#### Шаг 1 — Инициализация чарта

```bash
helm create myapp-chart
cd myapp-chart
```

Удалите лишнее из templates или адаптируйте под ваше приложение.

#### Шаг 2 — Адаптация Deployment

- Замените образ на ваш: `{{ .Values.image.repository }}:{{ .Values.image.tag }}`
- Добавьте envFrom из ConfigMap/Secret, если используете
- Параметризуйте resources

#### Шаг 3 — ConfigMap и Secret

- Создайте templates/configmap.yaml и templates/secret.yaml
- Данные берите из .Values.config и .Values.secrets (секреты — аккуратно, лучше через external secrets в проде)

#### Шаг 4 — Values для окружений

- values-dev.yaml: replicaCount=1, меньшие ресурсы, APP_ENV=dev
- values-prod.yaml: replicaCount=3, большие ресурсы, APP_ENV=prod

#### Шаг 5 — Установка и проверка

```bash
helm install myapp ./myapp-chart -f values-dev.yaml -n demo
helm upgrade myapp ./myapp-chart -f values-prod.yaml -n demo
helm rollback myapp 1 -n demo
```

#### Шаг 6 — README

- Описание основных параметров;
- Примеры: `helm install`, `helm upgrade` с разными values.

---

### 4. Acceptance criteria

- [ ] Чарт устанавливается без ошибок (`helm install`)
- [ ] Приложение работает после установки (проверка через curl/port-forward)
- [ ] Upgrade и rollback выполняются корректно
- [ ] Есть как минимум два values‑файла (dev/prod или аналог)
- [ ] README описывает параметры и примеры команд

---

### 5. Stretch goals

- Добавить Ingress как опцию (ingress.enabled)
- Подключить HPA через values
- Подготовить чарт для деплоя через Argo CD (структура Git‑репозитория)

---

См. `[[DevOps/04_Kubernetes/helm]]`, `[[DevOps/05_CI-CD/argocd]]`.
