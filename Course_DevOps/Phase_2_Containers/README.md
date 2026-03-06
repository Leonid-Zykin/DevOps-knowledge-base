## Phase 2 — Containers (Weeks 4–6)

В этой фазе вы переходите от работы «на голом Linux» к **контейнерам**:

- учитесь собирать образы Docker;
- запускаете контейнеры и управляете их сетью/томами;
- поднимаете многосервисные стенды с помощью Docker Compose.

> [!info] Цель фазы
> К концу Phase 2 вы должны уметь:
> - контейнеризировать простое приложение;
> - создать оптимальный и безопасный Dockerfile;
> - описать многосервисное приложение (app + DB + reverse proxy) в `docker-compose.yml`.

---

## Структура Phase 2

- `Module_04_Docker/`
  - `01_theory.md` — основы Docker, образы, контейнеры, сети и тома (`[[DevOps/03_Docker/docker-cheatsheet]]`, `[[DevOps/03_Docker/dockerfile-best-practices]]`, `[[DevOps/03_Docker/networking-and-volumes]]`, `[[DevOps/03_Docker/troubleshooting]]`).
  - `02_practice.md` — 15+ задач: сборка образов, работа с томами, сетями, отладка.
  - `03_project.md` — мини‑проект: контейнеризация реального приложения (Node.js/Python).
  - `04_quiz.md` — проверка понимания.

- `Module_05_Docker_Compose/`
  - `01_theory.md` — многосервисные приложения, сети и тома через Compose (`[[DevOps/03_Docker/docker-compose]]`, `[[DevOps/03_Docker/networking-and-volumes]]`).
  - `02_practice.md` — практика: multi‑service, env vars, healthcheck’и, override‑файлы.
  - `03_project.md` — мини‑проект: full‑stack стенд (app + DB + reverse proxy).
  - `04_quiz.md` — проверка понимания.

---

## Ожидаемые результаты

К концу Phase 2 вы:

- **Docker**
  - [ ] Понимаете разницу между образами, контейнерами, томами и сетями (`[[DevOps/03_Docker/docker-cheatsheet]]`).
  - [ ] Пишете многослойные Dockerfile с учётом best practices (`[[DevOps/03_Docker/dockerfile-best-practices]]`).
  - [ ] Умеете отлаживать проблемы контейнеров (`logs`, `exec`, `inspect`, `[[DevOps/03_Docker/troubleshooting]]`).

- **Compose**
  - [ ] Описываете многосервисный стенд через `docker-compose.yml` (`[[DevOps/03_Docker/docker-compose]]`).
  - [ ] Используете именованные тома и отдельные сети (`[[DevOps/03_Docker/networking-and-volumes]]`).
  - [ ] Настраиваете healthcheck’и и зависимости `depends_on`.

---

## Чек‑лист Phase 2

- [ ] Docker установлен и проверен (`docker run hello-world`, `docker compose version`).
- [ ] Выполнены все практики модуля `Module_04_Docker`, собран и отлажен хотя бы один рабочий образ приложения.
- [ ] Выполнены практики в `Module_05_Docker_Compose`, поднят многосервисный стенд (app + DB + reverse proxy).
- [ ] Мини‑проекты обоих модулей работают на вашей машине, есть заметки с командами запуска/остановки.
- [ ] Квизы пройдены с результатом не менее 80% правильных ответов.

