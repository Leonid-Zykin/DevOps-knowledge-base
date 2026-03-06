## Module 11 — CI Pipelines

### Почему это важно

CI (Continuous Integration) автоматизирует сборку, тесты и проверки при каждом коммите. Без CI баги накапливаются, релизы замедляются, ручные действия приводят к ошибкам. Middle DevOps должен уметь проектировать и поддерживать пайплайны.

Опираемся на:
- `[[DevOps/05_CI-CD/concepts]]`
- `[[DevOps/05_CI-CD/github-actions]]`
- `[[DevOps/05_CI-CD/gitlab-ci]]`
- `[[DevOps/05_CI-CD/jenkins]]`

---

## 1. CI/CD — базовые определения

Из `[[DevOps/05_CI-CD/concepts]]`:

- **CI** — автоматическая сборка, тестирование и проверка при каждом коммите/PR.
- **CD (Delivery)** — готовый к деплою артефакт, запуск в прод — по кнопке.
- **CD (Deployment)** — автоматический деплой в прод после проверок.
- **Pipeline** — последовательность стадий (lint, test, build, deploy).
- **Artifact** — результат сборки (образ, бинарь, Helm chart).

---

## 2. Типичная структура пайплайна

```text
stages:
  - lint      # форматирование, линтеры
  - test      # unit/integration тесты
  - build     # сборка образов/бинарей
  - security  # сканирование (опционально)
  - deploy    # деплой в окружения
```

---

## 3. GitHub Actions

Из `[[DevOps/05_CI-CD/github-actions]]`:

Workflows в `.github/workflows/*.yml`:

```yaml
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest -q
```

Secrets: Settings → Secrets → Actions. Использование: `${{ secrets.DEPLOY_TOKEN }}`.

---

## 4. GitLab CI

Из `[[DevOps/05_CI-CD/gitlab-ci]]`:

`.gitlab-ci.yml`:

```yaml
stages:
  - lint
  - test
  - build

lint:
  stage: lint
  script:
    - npm run lint
  only:
    - merge_requests

test:
  stage: test
  script:
    - npm test
  only:
    - merge_requests
    - main

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

---

## 5. Jenkins

Из `[[DevOps/05_CI-CD/jenkins]]`:

- Jenkinsfile (Declarative или Scripted Pipeline).
- Jobs: build, test, deploy.
- Плагины: Docker, Kubernetes, Git, credentials.

---

## 6. Артефакты и кэш

- **artifacts** — сохранять результаты job'ов для следующих стадий.
- **cache** — кэшировать зависимости (npm, pip, maven) для ускорения сборки.

---

## 7. Best practices

- Не хранить секреты в открытом виде; использовать secrets платформы.
- Разделять быстрые проверки (на каждый коммит) и тяжёлые (по расписанию/тегам).
- Деплой в stage и prod — одинаковый путь, разница только в конфиге.

---

## Что Middle DevOps должен знать по CI (чек‑лист)

- [ ] Понимаю структуру пайплайна: lint, test, build, deploy (`[[DevOps/05_CI-CD/concepts]]`).
- [ ] Умею писать workflow в GitHub Actions или GitLab CI (`[[DevOps/05_CI-CD/github-actions]]`, `[[DevOps/05_CI-CD/gitlab-ci]]`).
- [ ] Использую secrets для токенов и паролей.
- [ ] Настраиваю кэш и артефакты для ускорения сборки.
