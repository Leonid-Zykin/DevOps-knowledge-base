## Module 15 — DevSecOps

### Почему это важно

DevSecOps — встраивание проверок безопасности в каждый этап CI/CD, а не «в конце перед продом». Уязвимости в образах, зависимостях и коде должны обнаруживаться до деплоя. Middle DevOps должен уметь настраивать сканирование и управлять секретами.

Опираемся на:
- `[[DevOps/10_Security/devsecops]]`
- `[[DevOps/10_Security/secrets-management]]`
- `[[DevOps/10_Security/container-security]]`

---

## 1. SAST, DAST, сканирование

Из `[[DevOps/10_Security/devsecops]]`:

- **SAST** — статический анализ кода (Semgrep, SonarQube, Bandit).
- **DAST** — динамическое тестирование (OWASP ZAP, Burp).
- **Dependency scanning** — npm audit, pip-audit, trivy fs.
- **Container scanning** — Trivy, Grype для образов.

---

## 2. Trivy

```bash
trivy image my-registry/app:1.2.3
trivy fs .                    # файловая система
trivy config .                # конфиги (Dockerfile, k8s)
```

В CI: `trivy image --exit-code 1 --severity HIGH,CRITICAL ...`

---

## 3. Управление секретами

Из `[[DevOps/10_Security/secrets-management]]`:

- **Vault** — централизованное хранилище, динамические креды.
- **SOPS** — шифрование YAML/JSON (PGP, KMS).
- **SealedSecrets** — шифрование K8s Secret для Git.
- Не хранить секреты в Git, логах, образах.

---

## 4. Безопасность контейнеров

Из `[[DevOps/10_Security/container-security]]`:

- Минимальные базовые образы (alpine, distroless).
- Non-root: `runAsNonRoot: true`, `runAsUser: 1000`.
- `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`.
- `capabilities: drop: ["ALL"]`.

---

## 5. Security gates

- Блокировать merge при критичных уязвимостях.
- Или manual gate для деплоя при рисках.
- Процесс исключений с дедлайнами и владельцами.

---

## Что Middle DevOps должен знать (чек‑лист)

- [ ] Умею сканировать образы (Trivy) и зависимости (`[[DevOps/10_Security/devsecops]]`).
- [ ] Понимаю SOPS, SealedSecrets, Vault (`[[DevOps/10_Security/secrets-management]]`).
- [ ] Применяю securityContext в K8s (`[[DevOps/10_Security/container-security]]`).
