tags: [devops, docker]

## Docker Cheatsheet — основные команды

> ⚡ Tip: Почти для всех команд Docker есть формат `docker <resource> <verb>`: `docker container ls`, `docker image rm`, `docker volume inspect` и т.д.

### Образы

```bash
docker images                               # список образов
docker image ls                             # то же, new CLI
docker image ls repo/name                   # фильтр по имени

docker pull nginx:1.27                      # скачать образ
docker pull nginx                           # последний тег (latest)

docker image inspect nginx:1.27             # детали образа
docker history nginx:1.27                   # слои образа

docker image rm nginx:1.27                  # удалить образ
docker image prune                          # удалить dangling images
docker image prune -a                       # удалить неиспользуемые образы (ОСТОРОЖНО)
```

### Контейнеры: запуск и управление

```bash
docker ps                                   # запущенные контейнеры
docker ps -a                                # все контейнеры
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

docker run --name web -d -p 8080:80 nginx   # простой nginx
docker run --rm -it ubuntu:22.04 bash       # временный интерактивный контейнер
docker run --rm -v "$PWD":/work -w /work \
  python:3.11 python script.py              # однократный запуск

docker stop web                             # остановить
docker start web                            # запустить
docker restart web                          # перезапустить
docker rm web                               # удалить контейнер
docker rm -f web                            # удалить, даже если запущен
```

### Логи, exec и debug

```bash
docker logs web                             # логи контейнера
docker logs -f web                          # follow логов
docker logs --tail 100 web                  # последние 100 строк

docker exec -it web bash                    # зайти внутрь контейнера
docker exec web ls /usr/share/nginx/html    # выполнить команду

docker inspect web                          # подробная информация (JSON)
docker inspect -f '{{ .NetworkSettings.IPAddress }}' web
```

### Сети

```bash
docker network ls                           # список сетей
docker network inspect bridge               # дефолтная сеть

docker network create mynet                 # создать сеть
docker run --name db --network=mynet -d postgres:16
docker run --name api --network=mynet -d my-api:latest

docker network connect mynet web            # подключить контейнер к сети
docker network disconnect mynet web         # отключить
```

### Томa (volumes) и bind‑mount

```bash
docker volume ls                            # список томов
docker volume create data                   # создать том
docker volume inspect data                  # детали
docker volume rm data                       # удалить том
docker volume prune                         # удалить неиспользуемые (ОСТОРОЖНО)

docker run -d --name db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16                               # именованный том

docker run -d --name web \
  -v "$PWD"/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v "$PWD"/html:/usr/share/nginx/html:ro \
  nginx:1.27                                # bind-mount
```

### Копирование файлов

```bash
docker cp web:/etc/nginx/nginx.conf ./nginx.conf   # из контейнера на хост
docker cp ./index.html web:/usr/share/nginx/html/  # с хоста в контейнер
```

### Чистка ресурсов

```bash
docker container prune                      # удалить остановленные контейнеры
docker image prune                          # удалить dangling images
docker volume prune                         # удалить неиспользуемые тома
docker network prune                        # удалить неиспользуемые сети

docker system df                            # использование ресурсов
docker system prune                         # удалить неиспользуемые объекты
docker system prune -a                      # включая неиспользуемые образы (ОСТОРОЖНО)
```

### Build образов

```bash
docker build -t myapp:latest .              # собрать из текущего каталога
docker build -f Dockerfile.dev -t myapp:dev .
docker build --build-arg ENV=prod -t myapp:prod .
```

Многостейдж‑сборки и best practices см. в `[[dockerfile-best-practices]]`.

### Инспекция сети и соединений

```bash
docker exec web ss -tulpn                   # открытые порты внутри контейнера
docker exec web env                         # переменные окружения

docker inspect -f '{{json .NetworkSettings.Networks}}' web | jq
```

### docker-compose и стек из нескольких сервисов

```bash
docker compose up -d                        # поднять сервисы из docker-compose.yml
docker compose up -d api db                 # поднять только отдельные сервисы
docker compose ps                           # статус
docker compose logs -f api                  # логи сервиса
docker compose down                         # остановить и удалить ресурсы
```

Подробности см. в `[[docker-compose]]` и `[[networking-and-volumes]]`.

## Gotchas

- **Забытые тома и dangling images**  
  - **Проблема**: диск постепенно забивается старыми слоями и томами.  
  - **Решение**: регулярно смотреть `docker system df`, использовать `docker system prune` (с пониманием, что он удаляет).

- **Разница между томами и bind‑mount**  
  - **Проблема**: в dev удобно монтировать директории проекта, в prod это приводит к непредсказуемым правам и структурам.  
  - **Решение**: в проде по возможности использовать named volumes и декларативное управление (через `[[docker-compose]]` или Kubernetes).

- **`latest` теги**  
  - **Проблема**: образ `nginx:latest` может внезапно обновиться и сломать прод.  
  - **Решение**: всегда фиксировать версии (`nginx:1.27.2`), а `latest` использовать только локально.

- **`docker exec` вместо правильной отладки**  
  - **Проблема**: правки внутри контейнера теряются при пересборке, конфигурация расходится с Git.  
  - **Решение**: все изменения вносить в Dockerfile/compose/Helm (`[[dockerfile-best-practices]]`, `[[04_Kubernetes/helm]]`), а не править контейнер на лету.

> ⚠️ Warning: Команды `docker system prune -a` и массовое удаление томов/сетей на прод‑нодах без понимания, какие сервисы ими пользуются, может привести к потере данных и падению продакшена. Всегда проверяйте, какие ресурсы будут удалены, и имейте резервные копии (особенно для БД).

