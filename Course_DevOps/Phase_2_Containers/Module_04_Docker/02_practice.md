## Module 04 — Docker Practice

Ниже — 15 практических задач от базовых до продвинутых. Опирайтесь на:

- `[[DevOps/03_Docker/docker-cheatsheet]]`
- `[[DevOps/03_Docker/dockerfile-best-practices]]`
- `[[DevOps/03_Docker/networking-and-volumes]]`
- `[[DevOps/03_Docker/troubleshooting]]`

---

### Task 1 — Первый контейнер

- **Goal**: Запустить простой контейнер и понять `ps/logs`.
- **Steps**:
  1. Выполните:

```bash
docker run --name hello -d nginx:1.27
docker ps
docker logs hello
```

  2. Остановите и удалите контейнер.
- **Expected result**: Понимание цикла run → logs → stop → rm.
- **How to verify**:
  - `docker ps -a` показывает/не показывает контейнер согласно вашим действиям.

---

### Task 2 — Публикация порта и проверка

- **Goal**: Сделать сервис доступным с хоста.
- **Steps**:
  1. Запустите nginx:

```bash
docker run --name web -d -p 8080:80 nginx:1.27
```

  2. Проверьте:

```bash
ss -tulpn | grep 8080
curl -v http://127.0.0.1:8080
```

- **Expected result**: Доступ к nginx из браузера/через curl.
- **How to verify**:
  - Ответ nginx Welcome page на 8080;
  - порт виден в `docker ps` и `ss -tulpn`.

---

### Task 3 — Свой первый Dockerfile (одноступенчатый)

- **Goal**: Собрать простой образ статического сайта.
- **Steps**:
  1. Создайте `~/docker-lab/static-site` с `index.html`.
  2. Напишите Dockerfile на базе `nginx:1.27-alpine`, копирующий `index.html` в `/usr/share/nginx/html/`.
  3. Соберите образ `my-static:1.0` и запустите его на 8081 порту.
- **Expected result**: Ваш HTML отображается через контейнер nginx.
- **How to verify**:
  - `curl http://127.0.0.1:8081` возвращает ваш контент;
  - `docker images` показывает новый образ.

---

### Task 4 — Multi‑stage build для Node.js или Python (легкий вариант)

- **Goal**: Отделить этап сборки от этапа рантайма.
- **Steps**:
  1. Возьмите простое Node.js или Python‑API (например, hello‑world сервера).
  2. Напишите Dockerfile:
     - Stage 1: установка зависимостей, сборка (если нужно).
     - Stage 2: минимальный runtime (node:alpine / python:slim), копирование только нужного.
  3. Соберите образ, запустите контейнер, убедитесь, что endpoint `/` отвечает.
- **Expected result**: Рабочий multi‑stage образ.
- **How to verify**:
  - Образ собирается без ошибок;
  - `curl` к опубликованному порту отдаёт ответ приложения.

---

### Task 5 — Работа с томами: PostgreSQL

- **Goal**: Понять разницу между данными в контейнере и на томе.
- **Steps**:
  1. Создайте том `pgdata`:

```bash
docker volume create pgdata
```

  2. Запустите Postgres:

```bash
docker run -d --name pg \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16
```

  3. Подключитесь к БД (`psql`/cli) и создайте таблицу с данными.
  4. Удалите контейнер `pg` и запустите новый с тем же томом.
- **Expected result**: Данные сохраняются между перезапусками контейнера.
- **How to verify**:
  - Таблица и данные доступны после пересоздания контейнера.

---

### Task 6 — Bind mount для конфигов nginx

- **Goal**: Отделить конфиг от образа.
- **Steps**:
  1. Скопируйте дефолтный конфиг nginx:

```bash
mkdir -p ~/docker-lab/nginx-conf
docker run --rm nginx:1.27 cat /etc/nginx/nginx.conf > ~/docker-lab/nginx-conf/nginx.conf
```

  2. Измените конфиг (например, логируйте в другой файл или добавьте простой location).
  3. Запустите контейнер:

```bash
docker run --name web-conf -d \
  -p 8082:80 \
  -v "$HOME"/docker-lab/nginx-conf/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx:1.27
```

- **Expected result**: Nginx использует ваш конфиг.
- **How to verify**:
  - `curl`/логи отражают изменения из вашего nginx.conf.

---

### Task 7 — «Break it & fix it»: CrashLoop контейнера

- **Goal**: Научиться диагностировать контейнер, который сразу падает.
- **Steps**:
  1. Напишите Dockerfile для маленького скрипта, который завершается с кодом 1:

```bash
CMD ["sh", "-c", "echo 'I will crash'; exit 1"]
```

  2. Соберите образ и запустите его.
  3. Посмотрите:

```bash
docker ps -a
docker logs NAME
docker inspect NAME --format '{{.State.ExitCode}}'
```

  4. Исправьте CMD/скрипт так, чтобы контейнер не падал.
- **Expected result**: Вы видите, как выглядит «падающий» контейнер и как его чинить.
- **How to verify**:
  - Можете показать логи и код выхода до и после исправления.

---

### Task 8 — «Break it & fix it»: порты и сеть

- **Goal**: Потренироваться с неправильной публикацией портов.
- **Steps**:
  1. Запустите web‑контейнер **без** публикации порта:

```bash
docker run --name web-nopublish -d nginx:1.27
```

  2. Убедитесь, что:
     - `curl http://127.0.0.1` **не** видит nginx;
     - но `docker exec web-nopublish curl -v http://127.0.0.1` работает.
  3. Перезапустите контейнер с `-p 8083:80` и снова проверьте доступ.
- **Expected result**: Понимание разницы между портами внутри и снаружи.
- **How to verify**:
  - Можете объяснить, почему контейнер был доступен только изнутри.

---

### Task 9 — Docker inspect и диагностика env/volumes

- **Goal**: Научиться читать `docker inspect`.
- **Steps**:
  1. Запустите контейнер с несколькими переменными окружения и томами.
  2. Выполните:

```bash
docker inspect NAME | jq '.[0].Config.Env'
docker inspect NAME | jq '.[0].Mounts'
docker inspect NAME | jq '.[0].NetworkSettings.Networks'
```

- **Expected result**: Вы понимаете структуру `inspect` и где искать сетевые/томовые/ENV‑настройки.
- **How to verify**:
  - Можете показать, где в JSON‑структуре находятся нужные поля.

---

### Task 10 — Логи и их ротация

- **Goal**: Понять, куда пишутся логи контейнеров и как их ограничить.
- **Steps**:
  1. Запустите контейнер, который часто пишет в stdout.
  2. Найдите его log‑файл в `/var/lib/docker/containers/.../*-json.log`.
  3. Настройте лог‑драйвер и лимиты:

```bash
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  ...
```

- **Expected result**: Понимание, как избежать бесконтрольного роста лог‑файлов.
- **How to verify**:
  - Логи начинают ротироваться при достижении 10MB (можно ускорить, уменьшив размер).

---

### Task 11 — Использование healthcheck

- **Goal**: Добавить проверку здоровья контейнера.
- **Steps**:
  1. В Dockerfile для вашего web‑приложения добавьте:

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD \
  curl -f http://localhost/health || exit 1
```

  2. Соберите образ и запустите контейнер.
  3. Посмотрите:

```bash
docker ps
docker inspect NAME | jq '.[0].State.Health'
```

- **Expected result**: Статус health‑чека отображается в `docker ps` и `inspect`.
- **How to verify**:
  - При намеренном «ломании» `/health` статус становится `unhealthy`.

---

### Task 12 — Docker networking: своя bridge‑сеть

- **Goal**: Связать два контейнера по имени.
- **Steps**:
  1. Создайте сеть:

```bash
docker network create appnet
```

  2. Запустите БД и API в одной сети:

```bash
docker run -d --name db --network appnet postgres:16
docker run -d --name api --network appnet my-api:latest
```

  3. Внутри `api` проверьте доступ к `db` по имени:

```bash
docker exec -it api ping -c 1 db
```

- **Expected result**: Контейнеры видят друг друга по DNS‑имени.
- **How to verify**:
  - `ping db`/подключение к `db:5432` работают.

---

### Task 13 — Очистка ресурсов Docker

- **Goal**: Осознанно пользоваться командами prune.
- **Steps**:
  1. Посмотрите использование ресурсов:

```bash
docker system df
```

  2. Выполните dry‑run:

```bash
docker system prune --all --volumes --dry-run
```

  3. На тестовой машине (или в ограниченном виде) выполните `docker system prune` и посмотрите, что удалилось.
- **Expected result**: Понимание, какие ресурсы удаляются той или иной командой.
- **How to verify**:
  - Можете объяснить разницу между `container/image/volume/network prune` и `system prune`.

---

### Task 14 — Пример интеграции с CI (локальная симуляция)

- **Goal**: Подготовить команду сборки образа, пригодную для CI.
- **Steps**:
  1. В вашем приложении напишите скрипт `build_and_push.sh`, который:
     - собирает образ;
     - тегирует его с версией из аргумента;
     - пушит в выбранный registry (можно локальный/тестовый).
  2. Учтите:
     - переменные окружения для registry URL/логина;
     - аккуратную обработку ошибок.
- **Expected result**: Один скрипт, выполняющий типичную CI‑логику сборки образа.
- **How to verify**:
  - Скрипт успешно выполняется локально;
  - вы понимаете, как перенести его в GitLab CI/GitHub Actions (`[[DevOps/05_CI-CD/gitlab-ci]]`, `[[DevOps/05_CI-CD/github-actions]]`).

---

### Task 15 — «Break it & fix it»: права и non‑root

- **Goal**: Столкнуться с ошибками прав внутри контейнера и решить их.
- **Steps**:
  1. Напишите Dockerfile, где приложение пытается писать в каталог, к которому у пользователя нет прав (например, `/app/logs` с владельцем root и `USER app`).
  2. Запустите контейнер и посмотрите ошибки (`Permission denied`).
  3. Исправьте Dockerfile:
     - выставьте `chown`/`chmod` при сборке;
     - или измените путь записи.
- **Expected result**: Понимание типичных проблем с правами в контейнерах.
- **How to verify**:
  - После правок приложение успешно пишет логи.

---

## Итог

После выполнения всех задач вы должны:

- уверенно запускать и останавливать контейнеры;
- собирать образы (в т.ч. multi‑stage) и оптимизировать их;
- работать с томами и сетями;
- отлаживать типичные проблемы логами, inspect и exec;
- понимать последствия очистки ресурсов Docker.

Эти навыки напрямую будут использоваться в следующем модуле Docker Compose и при переходе к Kubernetes.

