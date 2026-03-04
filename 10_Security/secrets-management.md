tags: [devops, secrets]

## Secrets Management — Vault, SOPS, SealedSecrets, AWS Secrets Manager

> ⚡ Tip: Секреты никогда не должны храниться в Git/логах/образах. Используйте специализированные хранилища и шифрование.

### Общие принципы

- минимизировать количество людей/систем с доступом к секретам;
- использовать короткоживущие креды (STS/short‑lived tokens);
- централизованная ротация и аудит;
- отделять управление секретами от кода и конфигов.

### HashiCorp Vault (CLI‑пример)

Аутентификация:

```bash
vault login
vault status
vault secrets list
```

KV secrets:

```bash
vault secrets enable -path=kv kv-v2

vault kv put kv/app db_password=secret api_key=abc123
vault kv get kv/app

vault kv get -field=db_password kv/app
```

Динамические креды (пример PostgreSQL):

```bash
vault secrets enable database

vault write database/config/mydb \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@db:5432/postgres?sslmode=disable" \
  username="vault" \
  password="vault-password"

vault write database/roles/readonly \
  db_name=mydb \
  creation_statements="CREATE ROLE {{name}} WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO {{name}};" \
  default_ttl=1h \
  max_ttl=24h

vault read database/creds/readonly
```

### SOPS (PGP/KMS для YAML/JSON)

Зашифровать файл:

```bash
sops --encrypt --pgp <KEY_ID> secrets.yaml > secrets.enc.yaml
```

Расшифровать:

```bash
sops --decrypt secrets.enc.yaml > secrets.yaml
```

Пример `secrets.enc.yaml` в Git; ключи хранятся в KMS/PGP, не в репозитории.

### Sealed Secrets (Kubernetes)

Создание обычного Secret:

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=secret \
  --dry-run=client -o yaml > secret.yaml
```

SealedSecret:

```bash
kubeseal < secret.yaml > sealed-secret.yaml \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system

kubectl apply -f sealed-secret.yaml
```

В Git хранится только `SealedSecret`, который может расшифровать только контроллер в кластере.

### AWS Secrets Manager (CLI)

Создать секрет:

```bash
aws secretsmanager create-secret \
  --name app/db \
  --secret-string '{"user":"app","password":"secret"}'
```

Получить:

```bash
aws secretsmanager get-secret-value \
  --secret-id app/db \
  --query 'SecretString' \
  --output text
```

Связанные: `[[10_Security/network-security]]`, `[[10_Security/devsecops]]`, `[[04_Kubernetes/rbac-and-security]]`, `[[05_CI-CD/argocd]]`.

## Gotchas

- **Секреты в .env, Git, Slack**  
  - **Проблема**: утечки кредов при расшаривании логов/файлов.  
  - **Решение**: использовать сканеры секретов в CI, централизованные хранилища, ротацию при любых подозрениях.

- **Долгоживущие статические ключи**  
  - **Проблема**: украденный ключ даёт длительный доступ к системе.  
  - **Решение**: короткоживущие токены, STS, подписанные запросы, регулярная ротация.

- **Секреты в Docker образах и Terraform state**  
  - **Проблема**: даже после удаления значения остаются в слоях/state.  
  - **Решение**: исключать секреты из Dockerfile, шифровать/жёстко защищать state, использовать внешние data sources для секретов.

> ⚠️ Warning: Любой секрет, который хоть раз попал в публичное пространство (GitHub, публичный лог, issue) нужно считать безвозвратно скомпрометированным, даже если вы его «удалили». Единственно верный путь — немедленная ротация и аудит использования.

