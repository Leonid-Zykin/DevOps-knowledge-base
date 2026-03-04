tags: [devops, docker-dockerfile]

## Dockerfile — best practices (multi-stage, слои, ARG/ENV)

> ⚡ Tip: Чем меньше и более детерминированны слои Dockerfile, тем быстрее сборка и меньше риск неожиданных изменений.

### Базовый шаблон Dockerfile

```Dockerfile
FROM python:3.11-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "app:wsgi", "-b", "0.0.0.0:8000"]
```

### Multi-stage builds

```Dockerfile
# stage 1: build
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# stage 2: runtime
FROM nginx:1.27-alpine AS runtime
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
```

Преимущества:

- меньший конечный образ (без dev-зависимостей, исходников и билд‑инструментов);
- чёткое разделение стадий (build vs runtime).

### Слои и порядок инструкций

- Каждый `RUN`, `COPY`, `ADD` создаёт новый слой.
- Часто меняющиеся файлы должны быть внизу Dockerfile.

Плохо:

```Dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Хорошо:

```Dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

### ARG vs ENV

```Dockerfile
ARG APP_VERSION=dev
ENV APP_ENV=prod

RUN echo "Building version $APP_VERSION"
```

Передача `ARG` при build:

```bash
docker build --build-arg APP_VERSION=1.2.3 -t myapp:1.2.3 .
```

`ARG`:

- доступны только во время сборки;
- можно использовать для выбора зависимостей/базового образа.

`ENV`:

- остаются в конечном образе;
- используются приложением во время выполнения.

### Кэширование и репродюсируемость

```Dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
 && rm -rf /var/lib/apt/lists/*
```

- всегда очищать кеши менеджеров пакетов;
- фиксировать версии:

```Dockerfile
RUN pip install "fastapi==0.115.0" "uvicorn[standard]==0.30.0"
```

### Безопасность: пользователь, rootless, права

```Dockerfile
FROM node:20-alpine

RUN addgroup -S app && adduser -S app -G app
USER app
WORKDIR /app

COPY --chown=app:app . .
```

- по возможности НЕ запускать приложение от root;
- ограничивать права файлов/директорий.

### HEALTHCHECK

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD \
  curl -f http://localhost:8080/health || exit 1
```

Это используется оркестраторами (Docker, Swarm, Kubernetes) для проверки живости контейнера.

### Логи и stdout/stderr

- не писать логи в файлы внутри контейнера без явной причины;
- перенаправлять логи в stdout/stderr → дальше их собирает Docker/K8s/лог‑система.

Плохо:

```Dockerfile
CMD ["sh", "-c", "myapp > /var/log/app.log 2>&1"]
```

Хорошо:

```Dockerfile
CMD ["myapp"]
```

### Пример: Go multi-stage

```Dockerfile
FROM golang:1.22-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app ./cmd/app

FROM gcr.io/distroless/base-debian12 AS runtime
WORKDIR /app
COPY --from=build /src/app /app/app
USER nonroot:nonroot
ENTRYPOINT ["/app/app"]
```

### Пример: Python + gunicorn

```Dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN useradd -m app
USER app
WORKDIR /app

COPY --chown=app:app requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=app:app . .

CMD ["gunicorn", "app.wsgi:application", "-b", "0.0.0.0:8000", "--workers", "4"]
```

### Общие рекомендации

- фиксируйте базовые образы (`python:3.11-slim-bullseye`, а не просто `python:3.11`);
- минимизируйте количество слоёв (`RUN` объединяйте через `&&`);
- не храните секреты в Dockerfile/образах:
  - используйте переменные окружения, внешние хранилища (`[[secrets-management]]`);
- не используйте `ADD` без необходимости (tar/URL), в остальном — `COPY`.

Связанные разделы: `[[docker-cheatsheet]]`, `[[docker-compose]]`, `[[networking-and-volumes]]`, `[[04_Kubernetes/helm]]`.

## Gotchas

- **`latest` базовые образы**  
  - **Проблема**: обновление базового образа может разбить билд или безопасность.  
  - **Решение**: фиксировать теги и обновлять их контролируемо.

- **Кэш pip/apt внутри образа**  
  - **Проблема**: образы растут, кэш устаревает, сборка дольше.  
  - **Решение**: `--no-cache-dir` для pip, чистка `apt`/`apk` кэшей в том же слое.

- **Запуск приложения от root внутри контейнера**  
  - **Проблема**: при escape‑уязвимостях повышается ущерб.  
  - **Решение**: использовать небривилегированного пользователя (`USER app`) и ограничивать fs‑права.

- **Секреты в слоёном кеше**  
  - **Проблема**: даже если удалить файл с секретом в следующем `RUN`, он остаётся в предыдущем слое.  
  - **Решение**: не копировать секреты в образ; использовать build‑time решения (buildkit secrets), Vault, Kubernetes Secrets (`[[secrets-management]]`).

> ⚠️ Warning: Любые попытки «зачистить» секреты из уже запушенных образов (в реестрах Docker) должны считаться неуспешными: исходный слой мог быть скачан/закеширован сторонними системами. Считайте секрет скомпрометированным и отзывайте/ротуйте его.

