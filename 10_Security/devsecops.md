tags: [devops, devsecops]

## DevSecOps — SAST, DAST, dependency & container scanning

> ⚡ Tip: Цель DevSecOps — встроить проверки безопасности в каждый этап CI/CD, а не «в конце перед продом».

### SAST (Static Application Security Testing)

Примеры инструментов:

- SonarQube, Semgrep, Bandit, ESLint security плагины.

GitLab CI (пример):

```yaml
stages:
  - test
  - security

sast:
  stage: security
  image: returntocorp/semgrep
  script:
    - semgrep --config p/ci --error --json > semgrep-report.json
  artifacts:
    reports:
      sast: semgrep-report.json
    expire_in: 7 days
```

### Dependency scanning

Инструменты:

- `npm audit`, `pip-audit`, `safety`, `trivy fs`, `grype`.

Пример:

```bash
pip install pip-audit
pip-audit --progress-spinner=off --format=json > dep-report.json
```

### Container image scanning

CI‑пример:

```yaml
container_scan:
  stage: security
  image: aquasec/trivy
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL my-registry/app:$CI_COMMIT_SHA
```

### DAST (Dynamic Application Security Testing)

- OWASP ZAP, Burp Suite automation, GitLab DAST.

Пример (ZAP baseline scan):

```bash
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://staging.example.com \
  -r zap-report.html
```

### Security gates в пайплайне

- блокировать merge при:
  - наличии критичных уязвимостей;
  - неуспешном SAST/DAST/scan;
- или помечать как `manual` шаги деплоя, если есть риски.

Связанные: `[[05_CI-CD/gitlab-ci]]`, `[[05_CI-CD/github-actions]]`, `[[10_Security/secrets-management]]`, `[[10_Security/container-security]]`.

## Gotchas

- **Слишком много «красных» отчётов без действий**  
  - **Проблема**: команда перестаёт реагировать на отчёты SAST/сканеров.  
  - **Решение**: настроить приоритеты, фильтры, регрессионные политики и включать проверки поэтапно.

- **Блокирующие проверки без возможности исключений**  
  - **Проблема**: пайплайны стопорятся из‑за false positive/legacy.  
  - **Решение**: процесс оформления исключений с дедлайнами и владельцами, список approved‑risk.

- **Запуск DAST только против prod**  
  - **Проблема**: тесты безопасности сами по себе могут быть нагрузкой/риском.  
  - **Решение**: выделить staging/pre‑prod окружения с копией конфигурации и данных (обезличенных).

> ⚠️ Warning: Автоматизированные инструменты DevSecOps не заменяют ручного ревью и threat modeling. Отсутствие людей, понимающих результаты этих инструментов и принимающих решения, приводит либо к игнорированию алертов, либо к постоянным блокировкам develop‑процесса.

