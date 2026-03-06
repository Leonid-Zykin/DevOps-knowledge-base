## Module 04 — Docker Fundamentals

### Почему это важно

Контейнеры стали стандартным способом упаковывать приложения. DevOps‑инженер должен уметь:

- читать и писать Dockerfile;
- понимать, почему контейнер не запускается или падает;
- строить образы, которые быстро собираются, мало весят и безопасны.

Мы опираемся на:

- `[[DevOps/03_Docker/docker-cheatsheet]]`
- `[[DevOps/03_Docker/dockerfile-best-practices]]`
- `[[DevOps/03_Docker/networking-and-volumes]]`
- `[[DevOps/03_Docker/docker-compose]]`
- `[[DevOps/03_Docker/troubleshooting]]`

---

## 1. Ключевые понятия Docker

- **Image (образ)** — шаблон (root‑файловая система + метаданные), из которого создаются контейнеры.
- **Container (контейнер)** — запущенный экземпляр образа (процесс + namespaces/cgroups).
- **Layer (слой)** — результат каждой инструкции `RUN`, `COPY`, `ADD` в Dockerfile.
- **Registry** — хранилище образов (Docker Hub, ECR, GCR, GitHub Container Registry и т.д.).

Базовые команды из `[[DevOps/03_Docker/docker-cheatsheet]]`:

```bash
docker images
docker ps -a
docker run --rm -it ubuntu:22.04 bash
docker stop NAME
docker rm NAME
```

---

## 2. Dockerfile: из чего состоит образ

Простой Dockerfile (пример из `[[DevOps/03_Docker/dockerfile-best-practices]]`):

```Dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "app:wsgi", "-b", "0.0.0.0:8000"]
```

Ключевые инструкции:

- `FROM` — базовый образ;
- `WORKDIR` — рабочая директория;
- `COPY`/`ADD` — копирование файлов;
- `RUN` — выполнение команды при сборке (создаёт слой);
- `ENV` — переменные окружения в образе;
- `CMD`/`ENTRYPOINT` — команда, которая запускается при старте контейнера.

---

## 3. Multi‑stage build

Для уменьшения размеров и повышения безопасности используют многостадийную сборку:

```Dockerfile
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.27-alpine AS runtime
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
```

Преимущества (см. `[[DevOps/03_Docker/dockerfile-best-practices]]`):

- итоговый образ содержит только то, что нужно для запуска;
- нет dev‑зависимостей и исходников;
- меньше поверхность атаки.

---

## 4. Кэширование и порядок слоёв

Docker кэширует слои. Если инструкция не изменилась — слой берётся из кэша:

- **часто меняющееся** должно быть внизу Dockerfile;
- зависимости — выше, чтобы не перестраивать их при каждом изменении кода.

Плохо:

```Dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Лучше:

```Dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

---

## 5. ARG vs ENV

Из `[[DevOps/03_Docker/dockerfile-best-practices]]`:

```Dockerfile
ARG APP_VERSION=dev
ENV APP_ENV=prod

RUN echo "Building version $APP_VERSION"
```

- `ARG` — доступен **только на этапе сборки** (передаётся `--build-arg`);
- `ENV` — сохраняется в образе и доступен во время выполнения.

> [!warning] Не храните секреты в Dockerfile
> Любые секреты, попавшие в слои, остаются там навсегда (даже если вы удалили файл на следующем шаге). Используйте внешние секрет‑хранилища: `[[DevOps/10_Security/secrets-management]]`.

---

## 6. Запуск контейнеров: порты, окружение, тома

Базовые команды (см. `[[DevOps/03_Docker/docker-cheatsheet]]`):

```bash
docker run --name web -d -p 8080:80 nginx:1.27
docker run --rm -it ubuntu:22.04 bash
```

Где:

- `-p HOST:CONTAINER` — публикация порта;
- `-e KEY=VALUE` — переменные окружения;
- `-v` — тома и bind mount’ы.

Тома и bind mounts (`[[DevOps/03_Docker/networking-and-volumes]]`):

```bash
docker volume create data
docker run -d --name db \
  -e POSTGRES_PASSWORD=secret \
  -v data:/var/lib/postgresql/data \
  postgres:16
```

```bash
docker run -d --name web \
  -v "$PWD"/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v "$PWD"/html:/usr/share/nginx/html:ro \
  nginx:1.27
```

---

## 7. Сети Docker

Из `[[DevOps/03_Docker/networking-and-volumes]]`:

```bash
docker network create mynet
docker run -d --name db --network=mynet postgres:16
docker run -d --name api --network=mynet my-api:latest
docker exec api ping -c 1 db
```

Типы сетей:

- `bridge` — дефолтная изолированная сеть;
- `host` — контейнер использует сеть хоста (меньше изоляции);
- `none` — без сети;
- overlay — для Swarm (позже, в Kubernetes это решается CNI).

---

## 8. Отладка контейнеров

Минимальный чек‑лист из `[[DevOps/03_Docker/troubleshooting]]`:

1. Контейнер запущен?

```bash
docker ps -a
```

2. Что в логах?

```bash
docker logs NAME
docker logs -f NAME
```

3. Что внутри?

```bash
docker exec -it NAME bash
docker exec NAME env
docker exec NAME ss -tulpn
```

4. Сеть:

```bash
docker inspect NAME
docker network inspect NET
```

---

## 9. Безопасность контейнеров (базово)

Из `[[DevOps/10_Security/container-security]]`:

- **не запускать от root**, использовать `USER` в Dockerfile;
- минимизировать базовые образы (alpine/distroless);
- read‑only root filesystem;
- сканировать образы на уязвимости (Trivy, Grype).

Пример:

```Dockerfile
FROM node:20-alpine
RUN addgroup -S app && adduser -S app -G app
USER app
WORKDIR /app
COPY --chown=app:app . .
CMD ["node", "server.js"]
```

---

## 10. Связь с Kubernetes и CI/CD

- В Kubernetes Docker‑образы становятся **основой Pod’ов** (`[[DevOps/04_Kubernetes/core-concepts]]`).
- В CI/CD пайплайнах вы будете:
  - собирать и пушить образы (`[[DevOps/05_CI-CD/gitlab-ci]]`, `[[DevOps/05_CI-CD/github-actions]]`, `[[DevOps/05_CI-CD/jenkins]]`);
  - использовать Docker в тестах и проверках.

Docker‑навыки — фундамент для следующих фаз курса (Kubernetes, GitOps, security‑сканинг).

---

## Что Middle DevOps должен знать по Docker (чек‑лист)

- [ ] Понимаю разницу между образом, контейнером, томом и сетью.
- [ ] Умею читать и писать Dockerfile с учётом кэширования и best practices (`[[DevOps/03_Docker/dockerfile-best-practices]]`).
- [ ] Могу собрать и запустить образ, опубликовав порт и настроив переменные окружения.
- [ ] Умею монтировать тома и bind mount’ы, понимаю их различия (`[[DevOps/03_Docker/networking-and-volumes]]`).
- [ ] Могу отладить типичные проблемы контейнеров (CrashLoop, порты, сеть, права) по `docker logs/inspect/exec` (`[[DevOps/03_Docker/troubleshooting]]`).
- [ ] Знаю базовые практики безопасности контейнеров (non‑root, минимальные образы, отсутствие секретов в слоях) (`[[DevOps/10_Security/container-security]]`).

Если вы уверенно ставите галочки — теория усвоена, переходите к практическим заданиям модуля.

