## Module 11 — Mini-project: Full CI Pipeline

### 1. Цель проекта

**Задача**: построить полноценный CI-пайплайн для приложения:

- lint на каждый PR;
- unit-тесты;
- сборка Docker-образа;
- публикация образа в registry;
- опционально: деплой в dev/stage (если есть кластер).

Оценочное время: **4–8 часов**.

---

### 2. Требования

1. **Структура**:
   - Минимум 3 стадии: lint, test, build.
   - Отдельные jobs для каждой стадии.
2. **Lint**:
   - Форматирование и/или линтер для языка проекта.
3. **Test**:
   - Unit-тесты с отчётом о покрытии (опционально).
4. **Build**:
   - Docker build, push в registry.
   - Использование secrets для credentials.
5. **Условия**:
   - Lint на PR; build и push — только при merge в main (или по тегу).

---

### 3. Пошаговый план

#### Шаг 1 — Исходный проект

- Выберите приложение (Python/Node.js/Go) с тестами.
- Добавьте Dockerfile, если ещё нет.

#### Шаг 2 — Lint и test

- Настройте lint (flake8, eslint, go vet и т.п.).
- Добавьте job test с запуском тестов.

#### Шаг 3 — Build и push

- Job build: docker build, docker push.
- Registry: GHCR, GitLab Registry или Docker Hub.
- Secrets: REGISTRY_TOKEN, REGISTRY_USER.

#### Шаг 4 — Оптимизация

- Кэш зависимостей.
- Кэш слоёв Docker (buildx).
- paths: запускать только при изменении кода.

#### Шаг 5 — Документация

- README с описанием пайплайна и как его запускать локально.

---

### 4. Acceptance criteria

- [ ] Пайплайн запускается на push и PR
- [ ] Lint и test выполняются на PR
- [ ] Build и push — только при merge в main (или по тегу)
- [ ] Образ публикуется в registry
- [ ] Секреты не хранятся в открытом виде

---

### 5. Stretch goals

- Добавить security scanning (Trivy, Snyk).
- Деплой в dev-кластер через kubectl/helm (если есть доступ).
- Отчёты о покрытии (Codecov, Coveralls).

---

См. `[[DevOps/05_CI-CD/concepts]]`, `[[DevOps/05_CI-CD/github-actions]]`, `[[DevOps/05_CI-CD/gitlab-ci]]`.
