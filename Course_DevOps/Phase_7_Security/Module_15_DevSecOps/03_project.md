## Module 15 — Mini-project: Security Scanning in CI

### 1. Цель проекта

**Задача**: встроить security-проверки в CI-пайплайн:

- сканирование Docker-образа (Trivy);
- сканирование зависимостей (Trivy fs или npm audit / pip-audit);
- опционально: SOPS для секретов, securityContext в манифестах.

Оценочное время: **3–6 часов**.

---

### 2. Требования

1. **Container scanning**:
   - Job после build: `trivy image --exit-code 1 --severity HIGH,CRITICAL <image>`
   - Артефакт: отчёт (JSON/SARIF).
2. **Dependency scanning**:
   - Job: trivy fs или инструмент языка (npm audit, pip-audit).
   - Fail при критичных уязвимостях.
3. **Документация**:
   - README: как исправлять уязвимости, как оформлять исключения (.trivyignore).
4. **Опционально**:
   - SOPS для secrets в Git.
   - securityContext в Helm chart / K8s manifests.

---

### 3. Пошаговый план

#### Шаг 1 — Trivy image в CI

- Добавьте job после docker build/push.
- Используйте образ из registry с тегом $CI_COMMIT_SHA.
- --exit-code 1 при HIGH, CRITICAL.

#### Шаг 2 — Dependency scan

- trivy fs . или npm audit --audit-level=high.
- Сохраните отчёт как artifact.

#### Шаг 3 — Исключения

- Создайте .trivyignore для известных false positive.
- Каждое исключение — с комментарием и датой пересмотра.

#### Шаг 4 — Документация

- Как обновить зависимости.
- Как добавить исключение (процесс, кто утверждает).

---

### 4. Acceptance criteria

- [ ] CI выполняет container scan и dependency scan
- [ ] Пайплайн падает при критичных уязвимостях (или manual gate)
- [ ] Есть документация по исправлению и исключениям
- [ ] Опционально: секреты не в открытом виде (SOPS/Vault)

---

### 5. Stretch goals

- SARIF upload в GitHub Security / GitLab Security Dashboard.
- Автоматический PR с обновлением зависимостей (Dependabot, Renovate).
- Policy-as-code (Conftest, OPA) для K8s манифестов.

---

См. `[[DevOps/10_Security/devsecops]]`, `[[DevOps/10_Security/container-security]]`, `[[DevOps/10_Security/secrets-management]]`.
