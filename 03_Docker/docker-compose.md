tags: [devops, docker-compose]

## docker-compose — шпаргалка

> ⚡ Tip: Новый синтаксис CLI: `docker compose` (без дефиса). Старый `docker-compose` всё ещё часто встречается.

### Базовый пример `docker-compose.yml`

```yaml
version: "3.9"

services:
  db:
    image: postgres:16
    container_name: demo-db
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

  api:
    build: ./api
    container_name: demo-api
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/app
    ports:
      - "8080:8080"
    depends_on:
      - db
    networks:
      - backend

volumes:
  db-data:

networks:
  backend:
```

### Основные команды

```bash
docker compose up                 # запустить сервисы в foreground
docker compose up -d              # запустить в фоне
docker compose up -d api          # только сервис api

docker compose ps                 # статус сервисов
docker compose logs -f            # логи всех
docker compose logs -f api        # логи api

docker compose stop               # остановить
docker compose down               # остановить и удалить контейнеры/сети (по умолчанию без томов)
docker compose down -v            # + удалить тома

docker compose build              # собрать образы
docker compose build api          # собрать только api
docker compose pull               # стянуть образы
```

### Профили и override‑файлы

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

`docker-compose.override.yml` автоматически применяется поверх `docker-compose.yml`.

Пример overrides для dev:

```yaml
version: "3.9"
services:
  api:
    volumes:
      - ./api:/app:ro
    environment:
      APP_ENV: dev
```

### Переменные окружения и `.env`

- Можно использовать `.env` рядом с `docker-compose.yml`:

```env
POSTGRES_PASSWORD=secret
APP_ENV=prod
```

В compose:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

> ⚠️ Warning: Никогда не коммитьте `.env` с реальными прод‑секретами в Git. См. `[[secrets-management]]`.

### Здоровье сервисов и зависимости

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

### Networks и порты

```yaml
services:
  nginx:
    image: nginx:1.27
    ports:
      - "80:80"          # host:container
    networks:
      - frontend
      - backend

networks:
  frontend:
  backend:
```

См. также `[[networking-and-volumes]]` и `[[11_Networking/load-balancing]]`.

### Volumes

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

volumes:
  db-data:
```

### Override для prod

`docker-compose.prod.yml`:

```yaml
version: "3.9"
services:
  api:
    image: registry.example.com/app/api:1.2.3
    environment:
      APP_ENV: prod
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
```

Запуск:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Gotchas

- **`docker compose down -v` и потеря данных**  
  - **Проблема**: флаг `-v` удаляет тома (включая БД).  
  - **Решение**: использовать `-v` только в dev/ephemeral окружениях или при наличии бэкапов.

- **Разные версии Compose (v2 vs v3)**  
  - **Проблема**: не все ключи поддерживаются во всех версиях Docker/Compose.  
  - **Решение**: проверять документацию по версии Docker Engine и Compose; избегать swarm‑специфичных полей, если не используете Swarm.

- **Хардкод портов и конфликт с локальными сервисами**  
  - **Проблема**: `ports: "5432:5432"` конфликтует с локально установленным PostgreSQL.  
  - **Решение**: менять host‑порт (`15432:5432`), использовать `.env` с параметрами портов.

- **Смешивание секретов в файлах compose**  
  - **Проблема**: реальные пароли/ключи в `docker-compose.yml` попадают в Git.  
  - **Решение**: использовать переменные окружения и внешние секрет‑хранилища (`[[secrets-management]]`), а для прод‑оркестрации — Kubernetes/Swarm secrets.

> ⚠️ Warning: Использовать docker-compose как постоянную прод‑оркестрацию для критичных сервисов без мониторинга, health‑checks и автоперезапуска — рискованно. Для прод‑кластеров лучше использовать Kubernetes (`[[04_Kubernetes/kubectl-cheatsheet]]`, `[[04_Kubernetes/core-concepts]]`) или как минимум systemd‑юниты и мониторинг.

