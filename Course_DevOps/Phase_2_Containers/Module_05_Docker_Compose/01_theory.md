## Module 05 — Docker Compose & Multi‑service Apps

### Почему это важно

Реальные приложения редко состоят из одного сервиса. Типичный стенд: **app + DB + cache + reverse proxy + worker**. Docker Compose позволяет описать такой набор как один декларативный файл и запускать его одной командой.

Мы опираемся на:

- `[[DevOps/03_Docker/docker-compose]]`
- `[[DevOps/03_Docker/networking-and-volumes]]`
- `[[DevOps/03_Docker/troubleshooting]]`

---

## 1. Что такое Docker Compose

Compose описывает:

- **services** — контейнеры и их настройки;
- **volumes** — постоянные данные;
- **networks** — связи между сервисами.

Базовый пример из `[[DevOps/03_Docker/docker-compose]]`:

```yaml
version: "3.9"

services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

  api:
    build: ./api
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/app
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  db-data:
```

Запуск:

```bash
docker compose up -d
docker compose ps
docker compose logs -f api
```

---

## 2. Services: конфигурация контейнеров

Ключевые поля `services.<name>`:

- `image` / `build` — откуда взять образ;
- `ports` — публикация портов;
- `environment` / `env_file` — переменные окружения;
- `volumes` — тома/монтирования;
- `depends_on` — зависимости по запуску;
- `restart` — политика перезапуска;
- `healthcheck` — проверка здоровья.

Пример:

```yaml
services:
  api:
    image: my-registry/api:1.0.0
    ports:
      - "8080:8080"
    environment:
      APP_ENV: dev
      LOG_LEVEL: debug
    restart: unless-stopped
```

---

## 3. Volumes и хранение данных

Данные БД, очередей и т.п. должны жить в томах, а не внутри контейнера:

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Из `[[DevOps/03_Docker/networking-and-volumes]]`:

- named volumes — управляются Docker;
- bind mounts — прямые пути с хоста (`./config.yml:/app/config.yml:ro`).

> [!warning] `docker compose down -v`
> Флаг `-v` удаляет **тома** вместе со всем содержимым (включая БД). Используйте только в dev/ephemeral стендах или при наличии бэкапов.

---

## 4. Networks и изоляция сервисов

Compose создаёт отдельную сеть по умолчанию. Сервисы внутри сети видят друг друга по имени:

```yaml
services:
  api:
    networks:
      - backend
  db:
    networks:
      - backend

networks:
  backend:
```

Тогда `api` может обращаться к `db` по `db:5432`.  
Можно разделять front/back сети, как в примере из `[[DevOps/03_Docker/networking-and-volumes]]`.

---

## 5. Environment и .env

Compose поддерживает `.env` рядом с `docker-compose.yml`:

```env
POSTGRES_PASSWORD=secret
APP_ENV=dev
```

Использование:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

> [!warning] Не коммитьте продовые `.env` в Git
> Все секреты должны идти через менеджеры секретов/CI variables (`[[DevOps/10_Security/secrets-management]]`).

---

## 6. Healthcheck и depends_on

Базовый healthcheck:

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
```

Так `api` не стартует, пока `db` не станет `healthy`.

---

## 7. Override‑файлы и окружения

Compose позволяет накладывать несколько файлов:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

Типичный паттерн:

- `docker-compose.yml` — базовая конфигурация;
- `docker-compose.override.yml` — настройки для dev (автоматически подхватывается);
- `docker-compose.prod.yml` — продовые override’ы (образы из registry, лимиты ресурсов и т.п.).

Пример dev override (горячий код с хоста):

```yaml
services:
  api:
    volumes:
      - ./api:/app
    environment:
      APP_ENV: dev
```

---

## 8. Связь с CI/CD и Kubernetes

- В CI вы можете:
  - поднимать стенд для интеграционных тестов (`docker compose up -d api db`);
  - прогонять тесты и сворачивать (`docker compose down`).
- В Kubernetes часто переносите такую структуру в Helm‑чарт:
  - каждый service → Deployment + Service;
  - networks → namespace + k8s‑сети;
  - volumes → PV/PVC (`[[DevOps/04_Kubernetes/storage]]`).

---

## Что Middle DevOps должен знать по Compose (чек‑лист)

- [ ] Уверенно читаю и пишу `docker-compose.yml` с несколькими сервисами.
- [ ] Понимаю разницу между named volumes и bind mounts и использую их по назначению (`[[DevOps/03_Docker/networking-and-volumes]]`).
- [ ] Умею настраивать сети и проверять связанность сервисов.
- [ ] Использую `depends_on` и healthcheck’и для корректного порядка запуска.
- [ ] Понимаю, как разделять конфигурации dev/prod через override‑файлы (`[[DevOps/03_Docker/docker-compose]]`).
- [ ] Осознаю риски команд вроде `docker compose down -v` и не использую их бездумно на стендах с данными.

Если чек‑лист выполняется — переходите к практическим задачам модуля.

