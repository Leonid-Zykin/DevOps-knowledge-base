## Module 05 — Docker Compose Practice

Все задания выполняйте в отдельной директории, например `~/compose-lab`.  
Опирайтесь на `[[DevOps/03_Docker/docker-compose]]` и `[[DevOps/03_Docker/networking-and-volumes]]`.

---

### Task 1 — Минимальный compose: один сервис

- **Goal**: Запустить nginx через Compose.
- **Steps**:
  1. Создайте `docker-compose.yml`:

```yaml
version: "3.9"

services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
```

  2. Выполните `docker compose up -d` и проверьте:

```bash
docker compose ps
curl http://127.0.0.1:8080
```

- **Expected result**: Нginx доступен через 8080.

---

### Task 2 — API + DB в одной сети

- **Goal**: Запустить API и Postgres в одной сети через Compose.
- **Steps**:
  1. Используйте простой API‑образ (можно взять из проекта Module_04).
  2. Описать `api` и `db` в `docker-compose.yml`:
     - `db` — `postgres:16` c именованным томом;
     - `api` — `build: ./api` или `image: ...`, переменная `DATABASE_URL` указывает на `db:5432`.
  3. Запустите `docker compose up -d api db`.
- **Expected result**: API успешно подключается к БД по имени `db`.
- **How to verify**:
  - Логи `api` не содержат ошибок подключения;
  - `docker compose exec api ping -c 1 db` работает.

---

### Task 3 — Reverse proxy + API + DB

- **Goal**: Собрать трехсервисный стенд: `nginx` → `api` → `db`.
- **Steps**:
  1. Добавьте сервис `nginx`, который:
     - монтирует конфиг из `./nginx.conf`;
     - проксирует `/` на `http://api:8080`.
  2. Свяжите `nginx` и `api` в одной сети `frontend`; `api` и `db` — в `backend`.
- **Expected result**: Запросы на `http://127.0.0.1:80` идут через nginx к API.
- **How to verify**:
  - `curl -v http://127.0.0.1/health` проходит через reverse proxy;
  - `docker compose logs nginx` отражает запросы.

---

### Task 4 — Использование .env

- **Goal**: Вынести параметры в `.env`.
- **Steps**:
  1. Создайте `.env`:

```env
APP_PORT=8080
POSTGRES_PASSWORD=secret
```

  2. В `docker-compose.yml` используйте `${APP_PORT}` и `${POSTGRES_PASSWORD}`.
  3. Запустите стенд и убедитесь, что переменные подставились.
- **Expected result**: Параметры не хардкожены в compose‑файле.

---

### Task 5 — Healthcheck и depends_on

- **Goal**: Гарантировать, что API стартует только после готовности БД.
- **Steps**:
  1. Добавьте healthcheck к Postgres (см. теорию и `[[DevOps/03_Docker/docker-compose]]`).
  2. В `api` настройте:

```yaml
depends_on:
  db:
    condition: service_healthy
```

  3. Перезапустите стенд и посмотрите порядок старта по логам.
- **Expected result**: API не падает на старте из‑за неготовой БД.

---

### Task 6 — Override для dev

- **Goal**: Использовать override‑файл для dev‑режима.
- **Steps**:
  1. Создайте `docker-compose.override.yml`:

```yaml
version: "3.9"

services:
  api:
    environment:
      APP_ENV: dev
    volumes:
      - ./api:/app
```

  2. Запустите `docker compose up -d` (override применится автоматически).
- **Expected result**: В dev‑режиме код монтируется с хоста, можно быстро менять его без пересборки.

---

### Task 7 — Separate prod config

- **Goal**: Подготовить конфиг для «продоподобного» запуска.
- **Steps**:
  1. Создайте `docker-compose.prod.yml`, где:
     - `api.image` указывает на тег образа из registry;
     - включены лимиты ресурсов `deploy.resources.limits` (даже если они не применяются локально).
  2. Запустить:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

- **Expected result**: Вы понимаете, как отличать dev/prod конфигурации.

---

### Task 8 — «Break it & fix it»: неверный порт/хост

- **Goal**: Осознанно сломать и починить связь между сервисами.
- **Steps**:
  1. Временно измените `DATABASE_URL` на несуществующий хост или порт.
  2. Запустите стенд, посмотрите логи `api` — должны быть ошибки подключения.
  3. Почините конфиг, перезапустите и убедитесь, что всё заработало.
- **Expected result**: Понимание, как Compose‑конфиг влияет на сетевую связность.

---

### Task 9 — Просмотр состояния и логов

- **Goal**: Привыкнуть к `docker compose ps/logs/exec`.
- **Steps**:
  1. Запустите все сервисы.
  2. Попрактикуйтесь:

```bash
docker compose ps
docker compose logs -f api
docker compose logs -f db
docker compose exec api env
docker compose exec db psql -U app -d app -c '\dt'
```

- **Expected result**: Вы чувствуете себя уверенно при работе с Compose‑командами.

---

### Task 10 — Очистка стенда

- **Goal**: Понять разницу между `down` с/без томов.
- **Steps**:
  1. Заполните БД тестовыми данными.
  2. Выполните:

```bash
docker compose down
docker compose up -d
```

  3. Убедитесь, что данные **остались**.
  4. Затем:

```bash
docker compose down -v
docker compose up -d
```

  5. Убедитесь, что данные **пропали**.
- **Expected result**: Понимание, как `-v` влияет на данные.

---

## К усложнению (по желанию)

Если базовые задания даются легко:

- добавьте Redis/кеш как отдельный сервис и свяжите его с API;
- вынесите общие параметры в `x-` anchors YAML и переиспользуйте их;
- добавьте простой Prometheus + Grafana стек для метрик (будет полезно в Phase 6).

После выполнения практики вы будете готовы к мини‑проекту: full‑stack стенд с Compose.

