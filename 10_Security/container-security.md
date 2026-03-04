tags: [devops, container-security]

## Container Security — образы, non-root, read-only, Falco

> ⚡ Tip: Контейнер ≠ безопасность по умолчанию. Относитесь к нему как к обычному процессу на ноде, которому нужно ограничивать права.

### Образы и сканирование

- использовать минимальные базовые образы (alpine/distroless/ubi‑minimal);
- регулярно запускать сканирование уязвимостей:

```bash
trivy image my-registry/myapp:1.2.3
grype my-registry/myapp:1.2.3
```

Интеграция в CI: см. `[[10_Security/devsecops]]`.

### Non-root и права

Dockerfile:

```Dockerfile
RUN adduser -D -u 1000 app
USER app
WORKDIR /app

COPY --chown=app:app . .
```

Kubernetes:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Read-only root filesystem

```yaml
containers:
  - name: app
    image: myapp:1.2.3
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
      - name: tmp
        mountPath: /tmp
volumes:
  - name: tmp
    emptyDir: {}
```

### Falco (runtime security)

Фильтр угроз (пример):

```yaml
rules:
  - rule: Terminal shell in container
    desc: Detect interactive shell in container
    condition: >
      container.id != host and
      shell_procs and
      evt.type = execve
    output: >
      Terminal shell in container (user=%user.name container_id=%container.id command=%proc.cmdline)
    priority: WARNING
```

Связанные: `[[03_Docker/dockerfile-best-practices]]`, `[[04_Kubernetes/rbac-and-security]]`, `[[10_Security/devsecops]]`.

## Gotchas

- **Запуск контейнеров с `--privileged` и hostPath‑монтами**  
  - **Проблема**: почти полный доступ к хосту.  
  - **Решение**: использовать только для специально созданных утилит (backup/monitoring) и под строгим контролем.

- **Хранение секретов в образах**  
  - **Проблема**: даже после удаления они остаются в слоях и реестрах.  
  - **Решение**: секреты только через env/Secrets, внешние хранилища (`[[10_Security/secrets-management]]`).

- **Отсутствие ограничений ресурсов и securityContext**  
  - **Проблема**: контейнер может потреблять все ресурсы ноды, а уязвимость даст повышенные привилегии.  
  - **Решение**: всегда задавать `resources`, `securityContext`, PodSecurity уровни (`[[04_Kubernetes/rbac-and-security]]`).

> ⚠️ Warning: «Временные» контейнеры с root‑пользователем, монтирующие `/var/run/docker.sock` или `/` хоста для «удобной отладки», часто становятся постоянными и превращаются в критический вектор атаки. Ограничивайте подобные практики и используйте безопасные инструменты диагностики.

