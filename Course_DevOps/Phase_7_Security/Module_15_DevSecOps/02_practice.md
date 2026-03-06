## Module 15 — DevSecOps Practice

Нужны Trivy, SOPS (опционально), доступ к репозиторию с Dockerfile и CI.

Опирайтесь на `[[DevOps/10_Security/devsecops]]`, `[[DevOps/10_Security/secrets-management]]`, `[[DevOps/10_Security/container-security]]`.

---

### Task 1 — Trivy: сканирование образа

- **Goal**: Просканировать Docker-образ.
- **Steps**:
  1. Установите Trivy.
  2. Выполните `trivy image nginx:latest`
  3. Выполните `trivy image --severity HIGH,CRITICAL nginx:alpine`
  4. Сравните результаты для разных образов.
- **Expected result**: Понимание отчёта Trivy.

---

### Task 2 — Trivy: сканирование файловой системы

- **Goal**: Сканировать зависимости проекта.
- **Steps**:
  1. В проекте с lock-файлом (package-lock.json, requirements.txt) выполните `trivy fs .`
  2. Проверьте найденные уязвимости.
  3. Выполните `trivy config .` для Dockerfile.
- **Expected result**: Обнаружение уязвимостей в зависимостях и конфигах.

---

### Task 3 — Trivy в CI

- **Goal**: Добавить сканирование в пайплайн.
- **Steps**:
  1. Добавьте job в GitHub Actions / GitLab CI: trivy image после build.
  2. Используйте `--exit-code 1` при HIGH/CRITICAL.
  3. Сохраните отчёт как artifact.
- **Expected result**: Пайплайн падает при критичных уязвимостях.

---

### Task 4 — SOPS: шифрование файла

- **Goal**: Зашифровать секреты для Git.
- **Steps**:
  1. Установите SOPS, настройте PGP или age.
  2. Создайте `secrets.yaml` с паролями.
  3. Выполните `sops --encrypt secrets.yaml > secrets.enc.yaml`
  4. Проверьте, что secrets.enc.yaml можно коммитить.
  5. Расшифруйте: `sops --decrypt secrets.enc.yaml`
- **Expected result**: Секреты зашифрованы, хранятся в Git.

---

### Task 5 — SealedSecrets (Kubernetes)

- **Goal**: Создать SealedSecret из обычного Secret.
- **Steps**:
  1. Установите controller SealedSecrets в кластер.
  2. Создайте Secret: `kubectl create secret generic app-secret --from-literal=DB_PASSWORD=secret --dry-run=client -o yaml`
  3. Установите kubeseal, выполните `kubeseal -f secret.yaml -w sealedsecret.yaml`
  4. Примените sealedsecret.yaml — controller создаст Secret.
- **Expected result**: Secret в Git в зашифрованном виде.

---

### Task 6 — securityContext в Deployment

- **Goal**: Применить безопасные настройки.
- **Steps**:
  1. Добавьте в Deployment:
     - `runAsNonRoot: true`
     - `runAsUser: 1000`
     - `readOnlyRootFilesystem: true`
     - `allowPrivilegeEscalation: false`
     - `capabilities: drop: ["ALL"]`
  2. Проверьте, что приложение работает (может потребоваться emptyDir для /tmp).
  3. Если приложение требует root — разберитесь, как переписать под non-root.
- **Expected result**: Контейнер работает с ограниченными правами.

---

### Task 7 — Non-root в Dockerfile

- **Goal**: Собрать образ, работающий от non-root.
- **Steps**:
  1. Добавьте в Dockerfile: `RUN adduser -D app`, `USER app`, `COPY --chown=app:app`
  2. Соберите образ, запустите.
  3. Проверьте `docker run ... id` — должен быть non-root.
- **Expected result**: Образ не требует root.

---

### Task 8 — Исключения в Trivy

- **Goal**: Настроить игнор известных false positive.
- **Steps**:
  1. Создайте `.trivyignore` или `trivy.yaml` с исключениями (CVE, тип).
  2. Запустите trivy снова — исключённые уязвимости не должны блокировать.
  3. Документируйте каждое исключение с причиной и дедлайном.
- **Expected result**: Контролируемые исключения вместо полного отключения.

---

### Task 9 — Break it: секрет в логах

- **Goal**: Понять риск утечки.
- **Steps**:
  1. В CI добавьте `echo $DB_PASSWORD` (НЕ в проде с реальным паролем).
  2. Убедитесь, что платформа маскирует секрет.
  3. Удалите этот шаг.
- **Expected result**: Понимание, что секреты не должны попадать в логи.

---

### Task 10 — Интеграция: полный security gate

- **Goal**: Собрать в один пайплайн.
- **Steps**:
  1. Добавьте в CI: trivy image, trivy fs (или dependency scan).
  2. При HIGH/CRITICAL — fail.
  3. Документируйте процесс: как исправлять, как оформлять исключения.
- **Expected result**: Security как часть CI.

---

## Итог

После практики вы умеете:
- сканировать образы и зависимости Trivy;
- шифровать секреты (SOPS, SealedSecrets);
- применять securityContext в K8s;
- интегрировать security в CI.
