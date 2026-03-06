## Module 11 — CI Pipelines Practice

Практика на GitHub или GitLab (репозиторий с кодом). Jenkins — опционально, если есть доступ.

Опирайтесь на `[[DevOps/05_CI-CD/concepts]]`, `[[DevOps/05_CI-CD/github-actions]]`, `[[DevOps/05_CI-CD/gitlab-ci]]`.

---

### Task 1 — Минимальный workflow

- **Goal**: Запустить CI при push.
- **Steps**:
  1. Создайте репозиторий с простым проектом (Python/Node.js).
  2. Добавьте `.github/workflows/ci.yml` (или `.gitlab-ci.yml`) с одним job: checkout, install deps, run tests.
  3. Сделайте push и проверьте, что пайплайн выполняется.
- **Expected result**: Зелёный пайплайн при успешных тестах.

---

### Task 2 — Lint и test

- **Goal**: Добавить стадию lint.
- **Steps**:
  1. Добавьте lint (flake8, eslint, pylint и т.п.) как отдельный job или step.
  2. Добавьте unit-тесты (pytest, jest).
  3. Сделайте так, чтобы пайплайн падал при ошибках lint или тестов.
- **Expected result**: Два этапа проверки (lint + test).

---

### Task 3 — Build образа

- **Goal**: Собрать Docker-образ.
- **Steps**:
  1. Добавьте job для сборки образа (docker build).
  2. Запушьте образ в registry (GHCR, GitLab Registry, Docker Hub).
  3. Используйте secrets для логина в registry.
- **Expected result**: Образ публикуется при push в main.

---

### Task 4 — Кэш зависимостей

- **Goal**: Ускорить сборку.
- **Steps**:
  1. Добавьте кэш для pip/npm/maven (actions/cache или аналог).
  2. Сравните время сборки без и с кэшем.
- **Expected result**: Повторные сборки быстрее.

---

### Task 5 — Условный запуск

- **Goal**: Запускать только при изменении определённых путей.
- **Steps**:
  1. Настройте `paths` или `paths-ignore` в `on` (GitHub) или `rules` (GitLab).
  2. Запускайте сборку только при изменении `src/**` или `package.json`.
- **Expected result**: Пайплайн не запускается при изменении README и т.п.

---

### Task 6 — Matrix build

- **Goal**: Тестировать на нескольких версиях.
- **Steps**:
  1. Добавьте matrix (Python 3.10, 3.11, 3.12 или Node 18, 20).
  2. Запустите тесты для каждой версии.
- **Expected result**: Параллельные job'ы для разных версий.

---

### Task 7 — Артефакты между jobs

- **Goal**: Передать build-артефакт в следующую стадию.
- **Steps**:
  1. В job build сохраните артефакт (upload-artifact).
  2. В job deploy загрузите его (download-artifact) и используйте.
- **Expected result**: Артефакт передаётся между jobs.

---

### Task 8 — PR workflow

- **Goal**: Запускать CI на PR.
- **Steps**:
  1. Убедитесь, что workflow запускается на `pull_request`.
  2. Создайте PR и проверьте, что пайплайн выполняется.
  3. Добавьте branch protection: merge только при зелёном CI (если возможно).
- **Expected result**: CI на каждом PR.

---

### Task 9 — Break it: утечка секрета

- **Goal**: Понять важность маскирования.
- **Steps**:
  1. Добавьте step с `echo ${{ secrets.DEPLOY_TOKEN }}` (НЕ делайте в реальном проде).
  2. Убедитесь, что платформа маскирует значение в логах.
  3. Удалите этот step и никогда не используйте в проде.
- **Expected result**: Понимание, что секреты не должны попадать в логи.

---

### Task 10 — Break it: падающий тест

- **Goal**: Убедиться, что CI блокирует merge.
- **Steps**:
  1. Временно сломайте тест (assert False).
  2. Создайте PR — пайплайн должен падать.
  3. Восстановите тест.
- **Expected result**: CI предотвращает merge при падающих тестах.

---

## Итог

После практики вы умеете:
- писать workflow для lint, test, build;
- использовать secrets, кэш, артефакты;
- настраивать CI на push и PR.
