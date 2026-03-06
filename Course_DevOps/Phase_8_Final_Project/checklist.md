# Final Project — Чек-лист самопроверки

Пройдите чек-лист перед завершением проекта.

---

## Инфраструктура (Terraform)

- [ ] `terraform init`, `terraform plan`, `terraform apply` выполняются без ошибок
- [ ] Используются минимум 2 модуля (network, compute и т.п.)
- [ ] Переменные вынесены в tfvars для dev/prod
- [ ] Outputs выводят ключевые идентификаторы (vpc_id, cluster_endpoint и т.д.)
- [ ] `terraform destroy` корректно удаляет все ресурсы

---

## Kubernetes

- [ ] Приложение развёрнуто через Helm chart
- [ ] Deployment с replicas ≥ 2
- [ ] Service и Ingress/LoadBalancer настроены
- [ ] ConfigMap и Secret (через SealedSecrets/SOPS) используются
- [ ] Health checks (liveness, readiness) настроены
- [ ] securityContext применён (runAsNonRoot, readOnlyRootFilesystem, capabilities drop)

---

## CI/CD

- [ ] CI: lint, test, build, push образа в registry
- [ ] Trivy scan в CI (образ и/или зависимости)
- [ ] Argo CD Application создан и синхронизирован
- [ ] GitOps: изменение кода → CI → обновление Git → Argo CD sync → приложение обновлено
- [ ] Нет ручного `kubectl apply` / `helm install` для деплоя (всё через Git)

---

## Мониторинг

- [ ] Prometheus собирает метрики (node_exporter, приложение)
- [ ] Grafana подключена к Prometheus
- [ ] Минимум 2 дашборда (инфраструктура, приложение)
- [ ] Минимум 2 алерта (InstanceDown, HighCPU или аналог)
- [ ] Loki + Promtail собирают логи контейнеров
- [ ] Логи видны в Grafana с фильтрацией по namespace/pod

---

## Безопасность

- [ ] Trivy в CI, пайплайн падает при HIGH/CRITICAL (или documented exceptions)
- [ ] Секреты не хранятся в Git в открытом виде (SOPS, SealedSecrets, Vault)
- [ ] Образ собирается от non-root пользователя (USER в Dockerfile)
- [ ] securityContext в Pod ограничивает права

---

## Документация

- [ ] README в каждом репо: как развернуть, как запустить
- [ ] Архитектурная диаграмма (Mermaid) создана
- [ ] Описание алертов и как на них реагировать
- [ ] Процесс исключений для Trivy (если есть) документирован

---

## Итог

Если все пункты отмечены — проект готов к сдаче. Поздравляем с завершением курса «From Zero to Middle»!
