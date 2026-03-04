tags: [devops, github-actions]

## GitHub Actions — workflows, reusable actions, secrets

> ⚡ Tip: GitHub Actions запускает workflows из `.github/workflows/*.yml` на событиях (push, PR, schedule, release и т.д.).

### Минимальный workflow

`.github/workflows/ci.yml`:

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

### Основные элементы

- `on`: события триггера;
- `jobs`: набор задач, исполняющихся на runners;
- `runs-on`: тип runner'а (например, `ubuntu-latest`);
- `steps`: последовательность шагов (`run` или `uses`).

### Частые триггеры

```yaml
on:
  push:
    branches: [ main, develop ]
    paths:
      - "src/**"
  pull_request:
    types: [opened, synchronize, reopened]
  schedule:
    - cron: "0 3 * * *"          # UTC
  workflow_dispatch:              # ручной запуск
```

### Secrets и env

Настройка: Settings → Secrets and variables → Actions.

Использование:

```yaml
env:
  APP_ENV: prod

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      REGISTRY: ghcr.io
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploy to $APP_ENV"
      - run: echo "${{ secrets.DEPLOY_TOKEN }}" | docker login "$REGISTRY" --username user --password-stdin
```

> ⚠️ Warning: Никогда не выводите secrets (`${{ secrets.* }}`) в лог через `echo` — GitHub их маскирует, но утечка контента может остаться в сторонних системах (Slack, артефакты и т.п.).

### Reusable actions и composite actions

Использование готовых actions:

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: "20"
  - run: npm ci
  - run: npm test
```

Composite action в `.github/actions/deploy/action.yml`:

```yaml
name: "Deploy app"
description: "Deploy using kubectl"
runs:
  using: "composite"
  steps:
    - run: kubectl apply -f k8s/
      shell: bash
```

Использование в workflow:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/deploy
```

### matrix и параллельные job'ы

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pytest -q
```

### Артефакты и cache

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: actions/cache@v4
    with:
      path: ~/.cache/pip
      key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
      restore-keys: |
        ${{ runner.os }}-pip-

  - run: pip install -r requirements.txt

  - run: pytest -q

  - uses: actions/upload-artifact@v4
    with:
      name: dist
      path: dist/
```

### Примеры деплоя

- Docker → registry + kubectl/Helm:
  - см. `[[03_Docker/docker-cheatsheet]]`, `[[04_Kubernetes/kubectl-cheatsheet]]`, `[[04_Kubernetes/helm]]`;
- Terraform:
  - см. `[[06_Terraform/terraform-cheatsheet]]`, `[[06_Terraform/state-management]]`.

## Gotchas

- **Отсутствие ограничений для workflow событий**  
  - **Проблема**: workflow запускается на каждый push в любые ветки, включая POC/experiments.  
  - **Решение**: фильтровать по веткам/пути (`branches`, `paths`), использовать `if: github.ref == 'refs/heads/main'`.

- **Секреты в fork‑репозиториях**  
  - **Проблема**: по умолчанию секреты не доступны из fork'ов, и это хорошо; обход этого может открыть вектор атаки.  
  - **Решение**: не передавать secrets в неконтролируемые workflows, использовать environments и approval.

- **Долгие builds без cache**  
  - **Проблема**: каждый запуск заново тянет зависимости/собирает образы.  
  - **Решение**: настроить `actions/cache`, использовать кэш Docker‑слоёв (buildx cache, registry).

> ⚠️ Warning: GitHub Actions по сути исполняет произвольный код из вашего репозитория на runner'ах с доступом к секретам и внешним системам. Любой PR с изменениями в workflow'ах (особенно в public‑репо) должен проходить особо тщательное ревью, чтобы исключить утечки секретов и supply‑chain атаки.

