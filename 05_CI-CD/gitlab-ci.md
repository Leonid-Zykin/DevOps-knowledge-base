tags: [devops, gitlab-ci]

## GitLab CI — `.gitlab-ci.yml` шпаргалка

> ⚡ Tip: GitLab CI пайплайн = набор jobs, сгруппированных по stages; каждый job запускается на runner'е с заданным `image`/`tags`.

### Минимальный пример

```yaml
stages:
  - test
  - build

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

test:
  stage: test
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest -q

build:
  stage: build
  image: docker:27.0.3
  services:
    - docker:27.0.3-dind
  script:
    - docker build -t registry.example.com/app:$CI_COMMIT_SHA .
    - docker push registry.example.com/app:$CI_COMMIT_SHA
```

### Структура файла

- `stages`: порядок выполнения стадий;
- `variables`: глобальные переменные;
- `default`: настройки по умолчанию (image, before_script и т.п.);
- `job`'ы: блоки с `script`, `artifacts`, `only/except`/`rules`.

### Частые ключи job'а

```yaml
job-name:
  stage: test
  image: python:3.11
  script:
    - ...
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - .cache/pip
  only:
    - merge_requests
    - main
  tags:
    - docker
  needs:
    - build
```

### rules (современный способ условий)

```yaml
deploy:
  stage: deploy
  script: ./deploy.sh
  rules:
    - if: '$CI_COMMIT_TAG'
      when: manual
      allow_failure: false
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
    - when: never
```

### matrix/parallel

```yaml
tests:
  stage: test
  image: python:3.11
  parallel:
    matrix:
      - PY_VERSION: ["3.10", "3.11"]
        DB: ["postgres", "mysql"]
  script:
    - pyenv local "$PY_VERSION"
    - pytest -m "$DB"
```

### Include и шаблоны

```yaml
include:
  - local: .gitlab-ci-common.yml
  - project: devops/templates
    file: /.gitlab-ci-terraform.yml
  - remote: https://gitlab.com/example/common-ci/raw/main/common.yml
```

Anchors:

```yaml
.python-job: &python-job
  image: python:3.11
  before_script:
    - pip install -r requirements.txt

lint:
  <<: *python-job
  stage: test
  script: [flake8 .]
```

### Environments и deploy

```yaml
deploy_stage:
  stage: deploy
  script: ./deploy-stage.sh
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy_prod:
  stage: deploy
  script: ./deploy-prod.sh
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - tags
```

### Примеры интеграций

- Terraform: `[[06_Terraform/state-management]]`, `[[06_Terraform/best-practices]]`;
- Kubernetes (kubectl/Helm/Argo CD): `[[04_Kubernetes/kubectl-cheatsheet]]`, `[[04_Kubernetes/helm]]`, `[[argocd]]`.

## Gotchas

- **Смешивание `only/except` и `rules`**  
  - **Проблема**: неожиданный запуск или пропуск job'ов.  
  - **Решение**: для новых пайплайнов использовать только `rules`, не комбинировать с `only/except`.

- **Отсутствие кэша зависимостей**  
  - **Проблема**: долгие пайплайны из‑за повторной установки зависимостей.  
  - **Решение**: настраивать `cache` с аккуратными ключами (`$CI_COMMIT_REF_SLUG`, `$CI_JOB_NAME` и т.п.).

- **Секреты в `.gitlab-ci.yml`**  
  - **Проблема**: секреты попадают в Git и видны всем.  
  - **Решение**: использовать CI/CD variables (masked/protected) и внешние секрет‑хранилища (`[[10_Security/secrets-management]]`).

> ⚠️ Warning: Runner'ы GitLab CI, работающие на общих/неизолированных хостах, могут стать точкой компрометации для репозиториев и прод‑инфраструктуры. Ограничивайте, какие проекты могут использовать конкретные раннеры, и не запускайте непроверенный код с высокими правами на shared‑runner'ах.

