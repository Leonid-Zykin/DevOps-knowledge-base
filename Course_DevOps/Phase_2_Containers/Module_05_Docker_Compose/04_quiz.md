## Module 05 — Docker Compose Quiz

Проверьте себя по:

- `[[DevOps/03_Docker/docker-compose]]`
- `[[DevOps/03_Docker/networking-and-volumes]]`

---

### Вопрос 1 — Базовая команда

Какая команда поднимет все сервисы из `docker-compose.yml` в фоне?

a) `docker up -d`  
b) `docker compose up -d`  
c) `docker run -d`  
d) `docker compose start`

---

### Вопрос 2 — Связь сервисов

В `docker-compose.yml` два сервиса `api` и `db` описаны в одной сети. Как API обычно обращается к БД?

a) По `localhost`  
b) По имени сервиса `db` и порту 5432/3306  
c) Только по IP‑адресу  
d) Никак, нужны внешние DNS‑записи

---

### Вопрос 3 — Volumes

Что произойдёт с данными БД при выполнении:

```bash
docker compose down
```

если в `docker-compose.yml` тому БД соответствует именованный volume (`db-data`)?

a) Все данные БД будут удалены  
b) Данные сохранятся в volume и будут доступны при следующем `up`  
c) Удалятся только таблицы, но не схема  
d) Ничего не изменится, тома не поддерживаются в Compose

---

### Вопрос 4 — down -v

Чем отличается `docker compose down` от `docker compose down -v`?

a) Ничем, это синонимы  
b) `-v` дополнительно удаляет связанные тома (и данные в них)  
c) `-v` только удаляет контейнеры быстрее  
d) `-v` удаляет только сети

---

### Вопрос 5 — .env и переменные

Где Docker Compose по умолчанию ищет файл `.env` и как он используется?

a) В `/etc/docker/.env`, для настройки демона  
b) В текущей директории рядом с `docker-compose.yml`, подставляя значения `${VAR}`  
c) В домашней директории пользователя, не влияет на Compose  
d) Внутри контейнеров, как конфиг приложения

---

### Вопрос 6 — override‑файлы

Какой вызов корректно применит `docker-compose.yml` и `docker-compose.prod.yml` вместе?

a) `docker compose up docker-compose.yml docker-compose.prod.yml`  
b) `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d`  
c) `docker compose -p docker-compose.prod.yml up -d`  
d) Никак, Compose поддерживает только один файл

---

### Вопрос 7 — depends_on и healthcheck

Что делает конфигурация:

```yaml
depends_on:
  db:
    condition: service_healthy
```

в сервисе `api`?

a) Гарантирует, что контейнер `db` запустится до `api`, но не ждёт его готовности  
b) Гарантирует, что `api` стартует только после того, как healthcheck `db` станет `healthy`  
c) Означает, что `api` будет перезапускать `db` при ошибке  
d) Ничего, опция устарела

---

### Вопрос 8 — Просмотр логов

Какой командой лучше всего «подписаться» на логи сервиса `api` в текущем compose‑проекте?

a) `docker logs -f api`  
b) `docker compose logs -f api`  
c) `docker compose tail api`  
d) `docker tail -f api`

---

### Вопрос 9 — Сети

Если явно не указать сети в `docker-compose.yml`, что сделает Docker Compose?

a) Не создаст никакой сети, все сервисы будут недоступны  
b) Создаст общую сеть по умолчанию для всех сервисов этого проекта  
c) Подключит все сервисы к `bridge` сети Docker  
d) Создаст отдельную сеть для каждого сервиса

---

### Вопрос 10 — Практика dev/prod

Почему в проде не рекомендуется монтировать весь исходный код из хоста в контейнер через bind mount (`-v "$PWD":/app`)?

a) Это невозможно технически  
b) Это замедляет работу контейнера  
c) Это увеличивает поверхность атаки, усложняет контроль версий и может привести к неожиданным изменениям кода/конфигов  
d) Это автоматически делает контейнер read‑only

---

> [!note]- Answers
> **Вопрос 1**  
> b) `docker compose up -d`.  
>
> **Вопрос 2**  
> b) По имени сервиса `db` и порту БД.  
> Docker DNS в сети проекта резолвит имена сервисов.  
>
> **Вопрос 3**  
> b) Данные сохранятся в volume `db-data`.  
> `down` без `-v` не удаляет тома. См. `[[DevOps/03_Docker/docker-compose]]`.  
>
> **Вопрос 4**  
> b) `-v` дополнительно удаляет тома и данные.  
>
> **Вопрос 5**  
> b) В текущей директории, рядом с compose‑файлом.  
> Значения используются для подстановки `${VAR}`.  
>
> **Вопрос 6**  
> b) `docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d`.  
>
> **Вопрос 7**  
> b) Ждёт, пока `db` станет `healthy`, прежде чем запускать `api`.  
>
> **Вопрос 8**  
> b) `docker compose logs -f api`.  
>
> **Вопрос 9**  
> b) Создаёт общую сеть по умолчанию для проекта.  
>
> **Вопрос 10**  
> c) Увеличивает поверхность атаки и ломает принцип «immutable image». См. `[[DevOps/03_Docker/networking-and-volumes]]` и security‑рекомендации.  

