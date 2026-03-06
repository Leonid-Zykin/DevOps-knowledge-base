## Module 04 — Mini‑project: Dockerize a Real App

### 1. Цель проекта

**Задача**: взять реальное приложение (желательно небольшое API или веб‑приложение) на **Node.js или Python** и:

- упаковать его в Docker‑образ;
- настроить переменные окружения и логи;
- сделать образ достаточно «продовым»: multi‑stage, non‑root пользователь, healthcheck;
- задокументировать команды сборки и запуска.

Оценочное время: **4–8 часов**.

---

### 2. Требования к приложению

Можно использовать любое приложение:

- простой REST API (например, `/health`, `/api/items`);
- или небольшой веб‑сервер.

Минимальные требования:

- слушает HTTP‑порт (например, 8000);
- имеет endpoint `/health` (возвращает `200 OK` и простой JSON/текст);
- конфигурация (порт, debug‑флаги, строка подключения) может задаваться через переменные окружения.

---

### 3. Требования к контейнеризации

Ваш Docker‑образ должен:

1. **Собираться через multi‑stage build**:
   - Stage 1: установка зависимостей и сборка (если нужно);
   - Stage 2: минимальный runtime.
2. **Запускать приложение не от root**:

```Dockerfile
RUN adduser -D -u 1000 app
USER app
```

3. **Не содержать dev‑зависимостей** в финальном образе.
4. Иметь **HEALTHCHECK**, который проверяет `/health`:

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD \
  curl -f http://localhost/health || exit 1
```

5. Писать логи в stdout/stderr (без записи в файл внутри контейнера).
6. Быть параметризуемым через `ENV`/`ARG` (минимум: порт и режим окружения).

Опираться на best practices из `[[DevOps/03_Docker/dockerfile-best-practices]]` и `[[DevOps/10_Security/container-security]]`.

---

### 4. Пошаговый план

#### Шаг 1 — Подготовка исходного приложения

- Создайте директорию `~/projects/app-dockerized`.
- Поместите туда код приложения:
  - для Python — `requirements.txt`/`pyproject.toml` и `app.py`/ASGI/WSGI;
  - для Node.js — `package.json`, `src/`.
- Убедитесь, что локальный запуск без Docker работает:

```bash
python app.py         # или
node server.js
```

Проверьте endpoint `/health`:

```bash
curl http://127.0.0.1:8000/health
```

#### Шаг 2 — Базовый Dockerfile

Напишите первый (можно ещё не multi‑stage) Dockerfile и добейтесь:

- успешной сборки образа;
- работы контейнера с опубликованным портом:

```bash
docker build -t myapp:dev .
docker run --rm -d -p 8000:8000 --name myapp myapp:dev
curl http://127.0.0.1:8000/health
```

#### Шаг 3 — Multi‑stage и оптимизация

Перепишите Dockerfile в multi‑stage:

- Stage `build`:
  - устанавливает зависимости;
  - собирает артефакты (если нужно).
- Stage `runtime`:
  - использует минимальный базовый образ (alpine/slim/distroless);
  - копирует только необходимое из build‑стадии;
  - создаёт `app`‑пользователя и запускает приложение от него.

Сравните размер образов:

```bash
docker images | grep myapp
```

#### Шаг 4 — Переменные окружения и конфигурация

Сделайте так, чтобы:

- порт и environment могли задаваться через env‑переменные (например, `APP_PORT`, `APP_ENV`);
- значения по умолчанию задавались через `ENV` в Dockerfile;
- при запуске можно было переопределить их через `-e`.

Пример:

```bash
docker run -d -p 9000:9000 \
  -e APP_PORT=9000 \
  -e APP_ENV=prod \
  myapp:dev
```

Приложение должно реально использовать эти переменные.

#### Шаг 5 — Логи и healthcheck

- Убедитесь, что приложение пишет логи в stdout/stderr, а не в файл.
- Добавьте `HEALTHCHECK` в Dockerfile (см. выше).
- Проверьте:

```bash
docker ps
docker inspect myapp | jq '.[0].State.Health'
```

#### Шаг 6 — Обёртка в Makefile/скрипт (по желанию)

Для удобства создайте:

- `Makefile` с целями:

```Makefile
build:
	docker build -t myapp:$(TAG) .

run:
	docker run --rm -d -p 8000:8000 --name myapp myapp:$(TAG)

logs:
	docker logs -f myapp
```

или bash‑скрипт, автоматизирующий сборку/запуск.

---

### 5. Acceptance criteria

Проект считается завершённым, если:

1. **Dockerfile**:
   - multi‑stage (Build + Runtime);
   - в финальном образе нет dev‑зависимостей;
   - используются `ENV`/`ARG` по назначению;
   - контейнер запускается от non‑root пользователя.
2. **Работа контейнера**:
   - `docker run -d -p PORT:PORT myapp:<tag>` корректно запускает приложение;
   - endpoint `/health` возвращает 200;
   - HEALTHCHECK в Dockerfile показывает `healthy` при нормальной работе.
3. **Безопасность и best practices**:
   - логи идут в stdout/stderr;
   - нет секретов в Dockerfile/образе;
   - образ относительно небольшой (по сравнению с наивным вариантом).
4. **Документация**:
   - в `README.md`/заметке описаны:
     - зависимости (Node/Python версии);
     - команды сборки/запуска/остановки;
     - используемые переменные окружения.

---

### 6. Stretch goals

1. **Раздельные образы для prod/dev**  
   - `myapp:dev` с debug‑возможностями;  
   - `myapp:prod` — минимальный prod‑образ.

2. **Интеграция с CI**  
   - подготовить pipeline‑шаг (GitLab/GitHub/Jenkins) для:
     - сборки образа;
     - пуша в registry;
     - добавления тега вида `app:<git_sha>` и `app:<semver>`.

3. **Сканирование безопасности**  
   - прогнать образ через Trivy/Grype (см. `[[DevOps/10_Security/container-security]]`, `[[DevOps/10_Security/devsecops]]`);
   - зафиксировать основные результаты и потенциальные уязвимости.

---

### 7. Дальнейшее использование

- В следующем модуле Docker Compose вы добавите к этому образу:
  - базу данных;
  - reverse proxy (nginx);
  - локальный стенд из нескольких контейнеров.
- В фазе Kubernetes этот образ станет основой Deployment/StatefulSet.

Если вы уверенно можете собрать образ «с нуля» по описанию — вы прошли ключевой барьер в работе с контейнерами.

