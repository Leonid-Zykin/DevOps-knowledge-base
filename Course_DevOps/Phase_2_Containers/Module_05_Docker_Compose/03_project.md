## Module 05 — Mini‑project: Full‑stack App with Docker Compose

### 1. Цель проекта

**Задача**: собрать полноценный стенд из нескольких сервисов:

- backend API;
- база данных (PostgreSQL или MySQL);
- reverse proxy (nginx);
- (по желанию) frontend или статический сайт.

В результате у вас будет **одна команда** для запуска всего окружения разработки.

Оценочное время: **5–10 часов**.

---

### 2. Архитектура стенда

Текстовая схема:

```text
Client (browser/curl)
        |
        v
   [nginx reverse proxy]  :80
        |
        v
   [backend API]          :8080
        |
        v
   [DB: Postgres/MySQL]   :5432/3306
```

Обязательно:

- каждый компонент — отдельный сервис в `docker-compose.yml`;
- API использует БД для хранения данных (например, список задач/пользователей);
- nginx проксирует HTTP‑трафик к API по имени сервиса (`api`).

---

### 3. Требования к конфигурации

1. **Backend API**:
   - собирается из Dockerfile (можно использовать тот, что вы сделали в Module_04);
   - порт внутри контейнера фиксированный (например, 8080);
   - конфигурация БД через переменные окружения (`DATABASE_URL` или аналог).
2. **Database**:
   - использует официальный образ (`postgres:16`, `mysql:8` и т.п.);
   - хранит данные в именованном томе;
   - инициализируется пользователем/БД для приложения.
3. **nginx**:
   - конфиг монтируется через bind mount (`./nginx.conf:/etc/nginx/nginx.conf:ro`);
   - слушает 80 порт на хосте;
   - проксирует `/` к `api:8080`.
4. **Compose‑конфиг**:
   - как минимум две сети: `frontend` (nginx + api) и `backend` (api + db);
   - отдельный том для БД;
   - базовый healthcheck для БД и API;
   - dev‑override файл с монтированием кода приложения.

---

### 4. Пошаговый план

#### Шаг 1 — Подготовка проекта

- Создайте директорию `~/compose-fullstack`.
- Структура (пример):

```text
compose-fullstack/
  api/
    Dockerfile
    ...код приложения...
  nginx/
    nginx.conf
  docker-compose.yml
  docker-compose.override.yml
```

API‑часть можно взять из итогового проекта Module_04.

#### Шаг 2 — Описание БД

В `docker-compose.yml` опишите сервис `db`:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
```

И секцию volumes/networks:

```yaml
volumes:
  db-data:

networks:
  frontend:
  backend:
```

#### Шаг 3 — Описание API

Добавьте сервис `api`:

```yaml
services:
  api:
    build: ./api
    environment:
      APP_ENV: prod
      DATABASE_URL: postgres://app:secret@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend
```

Убедитесь, что API действительно читает `DATABASE_URL` и умеет выполнять миграции/инициализацию БД.

#### Шаг 4 — Healthcheck для БД и API

К `db` добавьте healthcheck (пример из теории и `[[DevOps/03_Docker/docker-compose]]`):

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U app -d app"]
  interval: 10s
  timeout: 5s
  retries: 5
```

К `api` — healthcheck по `/health` (можно через Dockerfile или напрямую в compose).

#### Шаг 5 — nginx reverse proxy

Создайте `nginx/nginx.conf`, который:

- слушает 80 порт;
- проксирует `/` на `http://api:8080`.

Сервис:

```yaml
services:
  nginx:
    image: nginx:1.27
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - frontend
```

#### Шаг 6 — dev‑override

Создайте `docker-compose.override.yml`:

```yaml
version: "3.9"

services:
  api:
    environment:
      APP_ENV: dev
    volumes:
      - ./api:/app
```

В dev‑режиме код API будет монтироваться с хоста, что удобно для разработки.

#### Шаг 7 — Тестирование стенда

- Запуск:

```bash
docker compose up -d
docker compose ps
```

- Проверки:

```bash
curl -v http://127.0.0.1/health
docker compose logs -f api
docker compose logs -f db
```

Убедитесь, что:

- API видит БД (`DATABASE_URL` работает);
- nginx корректно проксирует запросы;
- данные в БД сохраняются между перезапусками (`docker compose down` / `up`).

---

### 5. Acceptance criteria

Мини‑проект считается завершённым, если:

1. **Стенд стартует одной командой**:
   - `docker compose up -d` поднимает nginx + API + DB без фатальных ошибок;
   - `docker compose ps` показывает все сервисы в состоянии `running` (health — `healthy`, если настроен).
2. **Функциональность**:
   - `curl http://127.0.0.1/health` возвращает корректный ответ от API;
   - API реально использует БД (есть хотя бы один endpoint, который что‑то пишет/читает);
   - при `docker compose down` и `up` данные в БД сохраняются (благодаря именованному тому).
3. **Конфигурация**:
   - разделены сети `frontend` и `backend`;
   - есть именованный том `db-data`;
   - используются переменные окружения и/или `.env` для паролей и параметров;
   - есть dev‑override с монтированием кода.
4. **Документация**:
   - в `README.md` проекта описаны:
     - структура сервисов;
     - команды запуска/остановки;
     - переменные окружения;
     - известные ограничения.

---

### 6. Stretch goals

1. **Добавить Redis/кеш**  
   - сервис `redis`;
   - API использует его для кэширования данных или хранения сессий.

2. **Добавить Prometheus + Grafana (минимум)**  
   - поднять Prometheus и Grafana через Compose (`[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`);
   - собрать хотя бы базовые метрики контейнеров/приложения.

3. **Probes и rate‑limit в nginx**  
   - в `nginx.conf` настроить отдельный location `/health` без проксирования;
   - добавить простейший rate‑limit для API (подготовка к security‑модулям `[[DevOps/10_Security/network-security]]`).

---

### 7. Связь с дальнейшими фазами

- В фазе Kubernetes вы перенесёте этот стенд в кластер:
  - каждый сервис станет Deployment/StatefulSet + Service;
  - nginx — Ingress Controller или Ingress‑ресурс (`[[DevOps/04_Kubernetes/networking]]`).
- В фазе CI/CD — будете поднимать такой стенд в пайплайнах для интеграционных тестов (`[[DevOps/05_CI-CD/concepts]]`).
- В фазе Monitoring/Logging — подключите к нему Prometheus, Grafana, Loki.

Если вы уверенно поднимаете и диагностируете этот стенд, вы готовы к переходу на Kubernetes.

