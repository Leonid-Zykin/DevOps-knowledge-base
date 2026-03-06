## DevOps «From Zero to Middle» — дорожная карта

Этот курс рассчитан примерно на **20–21 неделю**, при нагрузке **8–12 часов в неделю**.  
Цель — довести человека с базовых навыков Linux до **уверенного Middle DevOps**, который умеет самостоятельно поднимать инфраструктуру, настраивать CI/CD, Kubernetes, мониторинг и безопасность.

> [!info] Как работать с курсом
> - Двигайтесь **строго по фазам**, не перепрыгивайте через фундаментальные модули.
> - В каждом модуле: сначала `01_theory.md`, затем `02_practice.md`, затем `03_project.md` и в конце `04_quiz.md`.
> - Все практики желательно выполнять **в своём окружении** (VM/WSL2 + локальный Kubernetes/облако).

---

## Общий обзор фаз

- **Phase 1 — Foundation (1–3 недели)**  
  Linux, сеть, Git. Вы уверенно работаете в терминале, понимаете, как устроены процессы, сеть и командная работа в Git.

- **Phase 2 — Containers (4–6 недели)**  
  Docker и Docker Compose. Умеете контейнеризировать приложение, собирать оптимальные образы и поднимать многосервисные стенды.

- **Phase 3 — Kubernetes (7–10 недели)**  
  Основы и продвинутая работа с K8s, Helm. Разворачиваете и сопровождаете приложения в кластере, понимаете RBAC, сети, storage.

- **Phase 4 — IaC (11–13 недели)**  
  Terraform и Ansible. Описываете инфраструктуру кодом, настраиваете серверы и сервисы идемпотентно.

- **Phase 5 — CI/CD (14–16 недели)**  
  GitLab CI, GitHub Actions, Jenkins, Argo CD. Строите полноценные пайплайны от коммита до деплоя в Kubernetes.

- **Phase 6 — Monitoring & Logging (17–18 недели)**  
  Prometheus, Grafana, Alertmanager, Loki/ELK. Настраиваете метрики, алерты, дашборды и централизованные логи.

- **Phase 7 — Security (19 неделя)**  
  DevSecOps, сканирование образов и зависимостей, управление секретами.

- **Phase 8 — Final Project (20–21 недели)**  
  Завершённый прод‑подобный сценарий: микросервисное приложение, инфраструктура, CI/CD, мониторинг, безопасность.

---

## Визуальный roadmap (Mermaid)

```mermaid
flowchart TD
  A[Старт<br/>Базовый Linux] --> P1

  subgraph P1[Phase 1 — Foundation (1–3 недели)]
    P1L[Module 01<br/>Linux<br/>[[DevOps/01_Linux/linux-cheatsheet]]<br/>[[DevOps/01_Linux/filesystem-and-permissions]]<br/>[[DevOps/01_Linux/processes-and-services]]<br/>[[DevOps/01_Linux/shell-scripting]]]
    P1N[Module 02<br/>Networking<br/>[[DevOps/01_Linux/networking]]<br/>[[DevOps/11_Networking/protocols]]<br/>[[DevOps/11_Networking/troubleshooting]]]
    P1G[Module 03<br/>Git<br/>[[DevOps/02_Git/git-cheatsheet]]<br/>[[DevOps/02_Git/branching-strategies]]<br/>[[DevOps/02_Git/hooks-and-automation]]]
    P1L --> P1N --> P1G
  end

  subgraph P2[Phase 2 — Containers (4–6 недели)]
    P2D[Module 04<br/>Docker<br/>[[DevOps/03_Docker/docker-cheatsheet]]<br/>[[DevOps/03_Docker/dockerfile-best-practices]]<br/>[[DevOps/03_Docker/networking-and-volumes]]]
    P2C[Module 05<br/>Docker Compose<br/>[[DevOps/03_Docker/docker-compose]]]
    P2D --> P2C
  end

  subgraph P3[Phase 3 — Kubernetes (7–10 недели)]
    P3B[Module 06<br/>K8s Basics<br/>[[DevOps/04_Kubernetes/core-concepts]]<br/>[[DevOps/04_Kubernetes/kubectl-cheatsheet]]]
    P3A[Module 07<br/>K8s Advanced<br/>[[DevOps/04_Kubernetes/workloads]]<br/>[[DevOps/04_Kubernetes/networking]]<br/>[[DevOps/04_Kubernetes/storage]]<br/>[[DevOps/04_Kubernetes/rbac-and-security]]]
    P3H[Module 08<br/>Helm<br/>[[DevOps/04_Kubernetes/helm]]]
    P3B --> P3A --> P3H
  end

  subgraph P4[Phase 4 — IaC (11–13 недели)]
    P4T[Module 09<br/>Terraform<br/>[[DevOps/06_Terraform/terraform-cheatsheet]]<br/>[[DevOps/06_Terraform/hcl-syntax]]<br/>[[DevOps/06_Terraform/modules]]<br/>[[DevOps/06_Terraform/state-management]]]
    P4A[Module 10<br/>Ansible<br/>[[DevOps/07_Ansible/ansible-cheatsheet]]<br/>[[DevOps/07_Ansible/playbooks-and-roles]]<br/>[[DevOps/07_Ansible/inventory]]]
    P4T --> P4A
  end

  subgraph P5[Phase 5 — CI/CD (14–16 недели)]
    P5CI[Module 11<br/>CI Pipelines<br/>[[DevOps/05_CI-CD/concepts]]<br/>[[DevOps/05_CI-CD/gitlab-ci]]<br/>[[DevOps/05_CI-CD/github-actions]]<br/>[[DevOps/05_CI-CD/jenkins]]]
    P5CD[Module 12<br/>CD & GitOps<br/>[[DevOps/05_CI-CD/argocd]]<br/>[[DevOps/04_Kubernetes/helm]]]
    P5CI --> P5CD
  end

  subgraph P6[Phase 6 — Monitoring (17–18 недели)]
    P6M[Module 13<br/>Prometheus & Grafana<br/>[[DevOps/08_Monitoring/prometheus]]<br/>[[DevOps/08_Monitoring/grafana]]<br/>[[DevOps/08_Monitoring/alertmanager]]]
    P6L[Module 14<br/>Logging<br/>[[DevOps/08_Monitoring/loki-and-logging]]<br/>[[DevOps/08_Monitoring/elk-stack]]]
    P6M --> P6L
  end

  subgraph P7[Phase 7 — Security (19 неделя)]
    P7S[Module 15<br/>DevSecOps<br/>[[DevOps/10_Security/secrets-management]]<br/>[[DevOps/10_Security/container-security]]<br/>[[DevOps/10_Security/devsecops]]]
  end

  subgraph P8[Phase 8 — Final Project (20–21 недели)]
    P8F[Итоговый проект<br/>Микросервисное приложение в облаке<br/>K8s + Terraform + CI/CD + Monitoring]
  end

  P1 --> P2 --> P3 --> P4 --> P5 --> P6 --> P7 --> P8
```

---

## Оценка по времени и результатам

- **Phase 1 — Foundation (1–3 недели, 25–35 часов)**  
  - Уверенное владение bash и инструментами Linux (`[[DevOps/01_Linux/linux-cheatsheet]]`, `[[DevOps/01_Linux/shell-scripting]]`).  
  - Понимание сетевых инструментов (`ping`, `traceroute`, `tcpdump`, `ss`) и iptables (`[[DevOps/01_Linux/networking]]`, `[[DevOps/11_Networking/troubleshooting]]`).  
  - Практический Git: ветвление, rebase, конфликт‑мерджи, хуки (`[[DevOps/02_Git/git-cheatsheet]]`, `[[DevOps/02_Git/branching-strategies]]`, `[[DevOps/02_Git/hooks-and-automation]]`).

- **Phase 2 — Containers (4–6 недели, 25–35 часов)**  
  - Умеете писать производительные и безопасные Dockerfile (`[[DevOps/03_Docker/dockerfile-best-practices]]`).  
  - Собираете и отлаживаете контейнеры, работаете с сетями и томами (`[[DevOps/03_Docker/docker-cheatsheet]]`, `[[DevOps/03_Docker/networking-and-volumes]]`).  
  - Поднимаете многосервисные окружения через Docker Compose (`[[DevOps/03_Docker/docker-compose]]`).

- **Phase 3 — Kubernetes (7–10 недели, 35–45 часов)**  
  - Понимаете core‑объекты и умеете управлять ними через `kubectl` (`[[DevOps/04_Kubernetes/core-concepts]]`, `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`).  
  - Настраиваете Deployment/StatefulSet, Service/Ingress, storage и RBAC (`[[DevOps/04_Kubernetes/workloads]]`, `[[DevOps/04_Kubernetes/networking]]`, `[[DevOps/04_Kubernetes/storage]]`, `[[DevOps/04_Kubernetes/rbac-and-security]]`).  
  - Используете Helm для упаковки и деплоя приложений (`[[DevOps/04_Kubernetes/helm]]`).

- **Phase 4 — IaC (11–13 недели, 25–35 часов)**  
  - Описываете инфраструктуру Terraform'ом (VPC/VM/базовая сеть) с модулями и remote state (`[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/modules]]`, `[[DevOps/06_Terraform/state-management]]`).  
  - Используете Ansible для конфигурации серверов и приложений (`[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`).

- **Phase 5 — CI/CD (14–16 недели, 25–35 часов)**  
  - Строите пайплайны в GitLab CI / GitHub Actions / Jenkins (`[[DevOps/05_CI-CD/gitlab-ci]]`, `[[DevOps/05_CI-CD/github-actions]]`, `[[DevOps/05_CI-CD/jenkins]]`).  
  - Понимаете концепции CI/CD и подход GitOps через Argo CD (`[[DevOps/05_CI-CD/concepts]]`, `[[DevOps/05_CI-CD/argocd]]`).

- **Phase 6 — Monitoring (17–18 недели, 20–25 часов)**  
  - Собираете метрики Prometheus, строите PromQL‑запросы, настраиваете алерты и дашборды (`[[DevOps/08_Monitoring/prometheus]]`, `[[DevOps/08_Monitoring/grafana]]`, `[[DevOps/08_Monitoring/alertmanager]]`).  
  - Централизуете логи через Loki/ELK (`[[DevOps/08_Monitoring/loki-and-logging]]`, `[[DevOps/08_Monitoring/elk-stack]]`).

- **Phase 7 — Security (19 неделя, 10–15 часов)**  
  - Управляете секретами, настраиваете сканирование образов и зависимостей, внедряете DevSecOps‑практики (`[[DevOps/10_Security/secrets-management]]`, `[[DevOps/10_Security/container-security]]`, `[[DevOps/10_Security/devsecops]]`).  
  - Понимаете основы сетевой безопасности и работы WAF/TLS (`[[DevOps/10_Security/network-security]]`).

- **Phase 8 — Final Project (20–21 недели, 30–40 часов)**  
  - Проект от идеи до прод‑подобного деплоя: инфраструктура, K8s, CI/CD, мониторинг, логирование, безопасность.

---

## Как понять, что вы уже ближе к Middle

- **Технично**:
  - Можете с нуля развернуть окружение: VM/облако → Docker/Compose → Kubernetes → мониторинг.  
  - Понимаете, как течёт запрос от браузера до БД (DNS → LB → Ingress → Service → Pod → DB/кэш).  
  - Умеете читать и чинить пайплайны CI/CD, Kubernetes‑манифесты, Terraform‑коды.

- **Практически**:
  - Не боитесь «ломать и чинить» стенд, при этом всегда имеете план отката.  
  - Для любой новой задачи сначала думаете об автоматизации и воспроизводимости (скрипт, playbook, Terraform, Helm), а не о ручных кликах.

- **Процессно**:
  - Понимаете базовые SRE‑концепции, умеете читать алерты и участвовать в разборе инцидентов (`[[DevOps/12_Soft-Skills-and-Processes/sre-concepts]]`, `[[DevOps/12_Soft-Skills-and-Processes/incident-response]]`).  
  - Можете аргументированно предложить улучшения по архитектуре и операционным практикам (`[[DevOps/12_Soft-Skills-and-Processes/architecture-patterns]]`).

