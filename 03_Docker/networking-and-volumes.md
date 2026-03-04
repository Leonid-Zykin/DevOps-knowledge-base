tags: [devops, docker-networking]

## Docker сети и тома (bridge, host, overlay, volume types)

### Типы сетей

```bash
docker network ls
```

Основные типы:

| Тип    | Описание                                  | Использование                          |
|--------|-------------------------------------------|----------------------------------------|
| bridge | дефолтная изолированная сеть на хосте    | одиночный хост, dev/prod               |
| host   | контейнер использует сеть хоста          | высокопроизводительный сетевой стэк   |
| none   | без сети                                 | jobs, оффлайн задачи                   |
| overlay| сеть между нодами (Swarm/K8s-like)       | multi-host (Swarm)                     |

### Bridge сети

Создание кастомной bridge‑сети:

```bash
docker network create mynet

docker run -d --name db --network=mynet postgres:16
docker run -d --name api --network=mynet my-api:latest

docker exec api ping -c 1 db         # имя контейнера как hostname
```

Инспекция сети:

```bash
docker network inspect mynet | jq '.[0].Containers'
```

### Host сеть

```bash
docker run --rm --network=host -it nginx:1.27
```

Особенности:

- контейнер разделяет сетевой стек с хостом;
- порты контейнера = порты хоста;
- упрощает интеграцию с локальными сервисами, но уменьшает изоляцию.

> ⚠️ Warning: `--network=host` в проде снижает сетевую изоляцию. Используйте только при необходимости (низкий latency, сложные сетевые настройки).

### Overlay (Swarm)

Для полноты:

```bash
docker swarm init
docker network create -d overlay myoverlay
```

В Kubernetes overlay‑сети реализуются CNI‑плагинами (см. `[[04_Kubernetes/networking]]`).

### Порты и публикация

```bash
docker run -d --name web -p 8080:80 nginx:1.27
```

- `8080` — порт на хосте;
- `80` — порт внутри контейнера.

Можно указывать IP:

```bash
docker run -d -p 127.0.0.1:8080:80 nginx
```

### Volumes vs Bind mounts

**Volumes (управляются Docker):**

```bash
docker volume create data
docker run -d -v data:/var/lib/postgresql/data postgres:16
```

**Bind mounts (директории хоста):**

```bash
docker run -d \
  -v "$PWD"/config.yml:/app/config.yml:ro \
  -v "$PWD"/logs:/var/log/app \
  myapp:latest
```

Сравнение:

| Тип        | Где хранится                 | Типичный use-case                   |
|-----------|------------------------------|-------------------------------------|
| Volume    | управляемое Docker хранилище | БД, постоянные данные в проде      |
| Bind      | конкретный путь на хосте     | dev, конфиги, логи, локальная отладка |

### Управление томами

```bash
docker volume ls
docker volume inspect data
docker volume rm data
docker volume prune
```

Поиск тома, привязанного к контейнеру:

```bash
docker inspect web | jq '.[0].Mounts'
```

### Монтирование конфигов и секретов

```bash
docker run -d \
  -v "$PWD"/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v "$PWD"/certs:/etc/nginx/certs:ro \
  nginx:1.27
```

> ⚠️ Warning: Никогда не монтируйте директорию репозитория целиком в проде (`-v "$PWD":/app`), если там есть `.git`, скрипты деплоя и т.п. — это увеличивает поверхность атаки и путаницу с версиями.

### Пример docker-compose с томами и сетями

```yaml
version: "3.9"

services:
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

  api:
    build: ./api
    networks:
      - backend
      - frontend

  nginx:
    image: nginx:1.27
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend

volumes:
  db-data:

networks:
  frontend:
  backend:
```

Связанные заметки: `[[docker-cheatsheet]]`, `[[docker-compose]]`, `[[11_Networking/load-balancing]]`, `[[04_Kubernetes/networking]]`.

## Gotchas

- **Случайное удаление томов с БД**  
  - **Проблема**: `docker volume prune` или `docker compose down -v` удаляют постоянные данные.  
  - **Решение**: явно маркировать критичные тома в документации, использовать бэкапы и не чистить тома на проде без плана восстановления.

- **Путаница с именами сетей и портами**  
  - **Проблема**: сервис недоступен, так как подключён не к той сети или порт не опубликован.  
  - **Решение**: всегда проверять `docker network inspect` и `docker ps` (колонка PORTS), пользоваться понятными именами сетей.

- **Bind‑mount поверх содержимого образа**  
  - **Проблема**: монтируемый каталог хоста скрывает файлы из образа (например, `/usr/share/nginx/html`).  
  - **Решение**: использовать bind только если точно нужно, иначе предпочитать named volumes и продуманные пути.

> ⚠️ Warning: Использование `--network=host` и широких bind‑mount'ов (`/:/host`) в контейнерах с root‑пользователем даёт почти полный доступ к хосту. Такие паттерны допустимы только для специальных утилит (backup‑агенты, мониторинг) и должны быть тщательно контролируемы.

