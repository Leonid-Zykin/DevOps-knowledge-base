tags: [devops, git-hooks]

## Git hooks и автоматизация (pre-commit, pre-push)

### Где живут хуки

- Локальные: `.git/hooks/*` (скрипты, запускаемые Git'ом).
- По умолчанию — примеры `*.sample`, нужно сделать их исполняемыми и убрать `.sample`.
- Часто хуки управляются через фреймворки (например, `pre-commit`).

```bash
ls .git/hooks
pre-commit.sample  pre-push.sample  commit-msg.sample  ...
```

> ⚡ Tip: Локальные хуки не версионируются сами по себе. Для шаринга с командой храните их в репо (например, `.githooks/`) и настраивайте `core.hooksPath`.

### Включение пользовательского каталога хуков

```bash
mkdir -p .githooks
git config core.hooksPath .githooks
```

Теперь Git будет искать хуки в `.githooks/` вместо `.git/hooks/`.

### Пример: pre-commit (быстрые проверки)

Файл: `.githooks/pre-commit`

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running pre-commit checks..."

# 1. Форматирование / линтер (пример для Python)
if command -v black >/dev/null 2>&1; then
  black .
fi

# 2. Проверка статических анализаторов
if command -v flake8 >/dev/null 2>&1; then
  flake8 .
fi

# 3. Запрет отладочных принтов / TODO
if git diff --cached | grep -E "print\(|console\.log|TODO"; then
  echo "Found debug prints or TODOs in staged changes."
  exit 1
fi

echo "pre-commit: OK"
```

Сделать исполняемым:

```bash
chmod +x .githooks/pre-commit
```

### Пример: pre-push (быстрые тесты)

Файл: `.githooks/pre-push`

```bash
#!/usr/bin/env bash
set -euo pipefail

remote="$1"
url="$2"

echo "pre-push: Testing before push to $remote ($url)"

if command -v pytest >/dev/null 2>&1; then
  pytest -q
fi

if command -v go >/dev/null 2>&1 && [ -f go.mod ]; then
  go test ./...
fi

echo "pre-push: OK"
```

### commit-msg hook (валидация сообщения коммита)

Файл: `.githooks/commit-msg`

```bash
#!/usr/bin/env bash
set -euo pipefail

MSG_FILE="$1"
MSG="$(head -n1 "$MSG_FILE")"

if [[ ${#MSG} -lt 10 ]]; then
  echo "Commit message too short."
  exit 1
fi

if ! echo "$MSG" | grep -Eq '^(feat|fix|chore|docs|refactor|test)(\(.+\))?:'; then
  echo "Commit message must follow Conventional Commits (feat:, fix:, ...)."
  exit 1
fi
```

### Использование `pre-commit` (фреймворк)

Установка:

```bash
pip install pre-commit
```

Файл `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/psf/black
    rev: 24.2.0
    hooks:
      - id: black
```

Установка хуков:

```bash
pre-commit install               # ставит hook для pre-commit
pre-commit run --all-files       # прогнать все хуки по всему репо
```

### Автоматизация на CI (GitLab CI / GitHub Actions)

GitLab CI (`.gitlab-ci.yml`):

```yaml
stages:
  - test

precommit:
  stage: test
  image: python:3.11
  script:
    - pip install pre-commit
    - pre-commit run --all-files
```

GitHub Actions (`.github/workflows/precommit.yml`):

```yaml
name: pre-commit
on: [push, pull_request]

jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install pre-commit
      - run: pre-commit run --all-files
```

Связанные заметки: `[[git-cheatsheet]]`, `[[05_CI-CD/gitlab-ci]]`, `[[05_CI-CD/github-actions]]`.

## Gotchas

- **Локальные хуки не в репозитории**  
  - **Проблема**: разные разработчики имеют разные проверки, часть багов не ловится до CI.  
  - **Решение**: хранить хуки в репо (`.githooks` или `pre-commit`), использовать `core.hooksPath` и документировать установку.

- **Слишком тяжёлые проверки в pre-commit**  
  - **Проблема**: долгие линтеры и тесты тормозят разработку.  
  - **Решение**: в pre-commit — только быстрые проверки; тяжёлые тесты перемещать в pre-push или CI.

- **Игнорирование exit-кодов**  
  - **Проблема**: hook всегда возвращает 0, даже при ошибках (например, `|| true`).  
  - **Решение**: использовать `set -euo pipefail` и не гасить ошибки без причины.

- **Отсутствие хуков на CI**  
  - **Проблема**: локальные проверки могут быть отключены/обойдены, а в CI этого нет.  
  - **Решение**: дублировать критичные проверки (линтеры, форматтеры, тесты) в пайплайнах CI (`[[05_CI-CD/concepts]]`).

> ⚠️ Warning: Никогда не полагайтесь только на локальные хуки для критичных проверок (например, проверка секретов, лицензий или security‑сканеры). Разработчики могут их отключить; все важные проверки должны обязательно выполняться в CI.

