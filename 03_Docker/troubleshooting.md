tags: [devops, docker-troubleshooting]

## Docker Troubleshooting — отладка контейнеров

> ⚡ Tip: При любых проблемах всегда начинайте с `docker ps`, `docker logs`, `docker inspect` и проверки сети (`docker network inspect`).

### Быстрый чек‑лист

1. Контейнер запущен? → `docker ps`.
2. Падает сразу после старта? → `docker logs <name>` и `docker inspect` (exitCode).
3. Порт слушается внутри? → `docker exec <name> ss -tulpn`.
4. Порт опубликован снаружи? → `docker ps` (колонка PORTS).
5. DNS/сеть? → `docker exec <name> ping`, `curl`, `dig`.

### Контейнер сразу завершается

```bash
docker ps -a
docker logs <container>
docker inspect <container> --format '{{.State.ExitCode}} {{.State.Error}}'
```

Типичные причины:

- неправильная команда `CMD`/`ENTRYPOINT`;
- ошибки в конфиге приложения;
- отсутствие зависимостей (БД, очередь и т.п.).

Воспроизведение ошибки интерактивно:

```bash
docker run --rm -it --entrypoint=/bin/sh image:tag
```

### Проблемы с сетью и портами

**Внутри контейнера:**

```bash
docker exec -it web ss -tulpn              # слушает ли сервис нужный порт
docker exec -it web curl -v http://127.0.0.1:8080/health
docker exec -it web env | grep PORT
```

**Снаружи:**

```bash
docker ps                                  # колонка PORTS
curl -v http://127.0.0.1:8080/health
```

Если используется compose → см. `[[docker-compose]]` и `[[networking-and-volumes]]`.

### Отладка DNS / внешних сервисов

```bash
docker exec -it api ping -c 3 db
docker exec -it api dig db
docker exec -it api curl -v http://db:5432
```

Проверка `/etc/hosts` и `resolv.conf`:

```bash
docker exec api cat /etc/hosts
docker exec api cat /etc/resolv.conf
```

### Доступ к файлам и правам

```bash
docker exec api ls -lah /app
docker exec api whoami
docker exec api id
docker inspect api | jq '.[0].Config.User'
```

Распространённые случаи:

- контейнер работает не от `root` и не имеет прав на каталог/файл (`Permission denied`);
- bind‑mount монтируется с правами хоста, которые не подходят пользователю внутри.

Решения:

- исправить владельца/права на хосте (`chown`, `chmod`);
- в Dockerfile использовать `USER` и `COPY --chown=...` (`[[dockerfile-best-practices]]`).

### Диск и память

```bash
docker system df                       # использование диска
docker stats                           # текущий расход CPU/RAM

docker inspect <container> | jq '.[0].HostConfig.Memory'
```

При OOM‑киллах:

- смотреть `dmesg`/логи ядра;
- уменьшать потребление памятью или повышать лимиты (в compose/K8s).

### Сбор мусора и утечки

```bash
docker ps -a
docker container prune                 # удалить остановленные
docker image prune                     # удалить dangling images
docker volume ls
docker volume prune                    # удалить неиспользуемые тома
``>

> ⚠️ Warning: Прежде чем удалять тома (`volume prune`) на проде, убедитесь, какие БД/сервисы ими пользуются. Потеря тома с БД = потеря данных.

### Логи и ротация

Проверка размера логов:

```bash
docker inspect <container> | jq '.[0].HostConfig.LogConfig'
ls -lh /var/lib/docker/containers/*/*-json.log
```

Настройка лог‑драйвера и лимитов (пример для json‑file):

```bash
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  nginx:1.27
```

В `daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5"
  }
}
```

Связанные разделы: `[[docker-cheatsheet]]`, `[[dockerfile-best-practices]]`, `[[04_Kubernetes/troubleshooting]]`.

## Gotchas

- **Трудно воспроизводимые баги внутри контейнера**  
  - **Проблема**: ручные правки в контейнере не попадают в Dockerfile, баги исчезают и появляются.  
  - **Решение**: любые фиксы делать через Dockerfile/compose/Helm и версионировать в Git.

- **Неочевидные ограничения ресурсов Docker daemon'а**  
  - **Проблема**: контейнеры падают или тормозят из‑за лимитов CPU/RAM/IO, выставленных в Docker Desktop/daemon.  
  - **Решение**: проверять настройки daemon'а, использовать мониторинг (`[[08_Monitoring/prometheus]]`, `[[08_Monitoring/grafana]]`).

- **Неправильный контекст build'а**  
  - **Проблема**: в образ попадают лишние файлы (секреты, кэш), сборка долгая и нестабильная.  
  - **Решение**: использовать `.dockerignore`, минимизировать контекст (`docker build -f path/Dockerfile path/ctx`), смотреть `docker build` вывод.

> ⚠️ Warning: Отладка в проде через `docker exec` допустима только как временная мера. Не оставляйте открытые шеллы, не меняйте файлы напрямую в prod‑контейнерах и не используйте это как постоянный способ администрирования — это ломает принцип immutable infrastructure.

