## Пререквизиты и подготовка окружения

Этот курс рассчитан на людей с **базовыми знаниями Linux** (умеете зайти по SSH, открыть терминал, выполнять простые команды). Всё остальное вы получите по ходу.

> [!tip] Если что‑то кажется незнакомым
> Не останавливайтесь. Отметьте себе термин/команду и вернитесь к нему по ссылкам в базе знаний `[[DevOps/01_Linux/linux-cheatsheet]]` и соседним заметкам.

---

## Минимальные технические требования

- **Железо**:
  - 4 CPU ядра (лучше 6+);
  - 16 ГБ RAM (8 ГБ — нижний предел, придётся аккуратно работать с Kubernetes);
  - 100+ ГБ свободного места на диске.

- **ОС хоста**:
  - Linux (желательно Ubuntu 22.04+) **или**
  - Windows 11/10 Pro с поддержкой **WSL2** **или**
  - macOS (M1/M2/M3 или Intel) — подойдёт, но примеры будут ориентированы на Linux.

---

## Варианты окружения

- **Вариант 1 — Linux как основная ОС (рекомендуется)**  
  Всё делаете прямо на своём Linux‑ноутбуке/сервере.

- **Вариант 2 — Windows + WSL2**  
  Устанавливаете Ubuntu в WSL2 и работаете там, как в обычном Linux.

- **Вариант 3 — VM в гипервизоре**  
  - VirtualBox / VMware / Hyper-V с Ubuntu 22.04 LTS;
  - выделяете VM не менее: 4 CPU, 8–12 ГБ RAM, 80+ ГБ диска.

---

## Базовый набор инструментов

Перед началом курса установите:

- Терминал и SSH‑клиент:
  - Linux/macOS: уже есть (`ssh`, `tmux`, `screen`);
  - Windows: Windows Terminal + OpenSSH Client / PuTTY (лучше WSL2 и использовать Linux‑ssh).

- Редактор кода:
  - VS Code / Cursor / JetBrains IDE;
  - включите поддержку YAML, Dockerfile, Terraform, Ansible.

- Git:
  - установите Git и настройте имя/почту:

```bash
git config --global user.name "Ваше Имя"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

См. шпаргалку `[[DevOps/02_Git/git-cheatsheet]]`.

---

## Инструменты по фазам

### Phase 1 — Foundation

Обязательно:

- `bash`, `ssh`, `tmux` или `screen`;
- утилиты: `curl`, `wget`, `git`, `htop`, `jq`, `net-tools` (или `iproute2`), `traceroute`, `tcpdump`.

На Ubuntu:

```bash
sudo apt update
sudo apt install -y git curl wget htop jq net-tools traceroute tcpdump tmux
```

Для углубления по Linux используйте: `[[DevOps/01_Linux/linux-cheatsheet]]`, `[[DevOps/01_Linux/processes-and-services]]`, `[[DevOps/01_Linux/networking]]`.

### Phase 2 — Containers (Docker + Compose)

Нужно установить:

- Docker Engine;
- Docker Compose (современный `docker compose` уже в составе Docker).

Подробное пошаговое руководство будет в `Resources/tools-installation.md`, но в общих чертах на Ubuntu:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker "$USER"
newgrp docker

docker version
docker info
```

Проверьте:

```bash
docker run --rm hello-world
docker compose version
```

См. `[[DevOps/03_Docker/docker-cheatsheet]]`, `[[DevOps/03_Docker/docker-compose]]`.

### Phase 3 — Kubernetes (minikube / kind)

Вам потребуется локальный кластер:

- **minikube** или **kind** (kubernetes‑in‑docker);
- `kubectl` CLI;
- для Helm‑модуля — `helm`.

Рекомендуемый стек:

- `kubectl` — по шпаргалке `[[DevOps/04_Kubernetes/kubectl-cheatsheet]]`;
- `minikube` или `kind` — инструкции в `Resources/tools-installation.md`;
- `helm` — по `[[DevOps/04_Kubernetes/helm]]`.

Минимальная проверка:

```bash
kubectl version --client
kubectl get nodes
```

Для локального кластера (пример с minikube):

```bash
minikube start --cpus=4 --memory=8192 --driver=docker
kubectl get nodes
```

### Phase 4 — IaC (Terraform + Ansible)

Нужно:

- **Terraform CLI** (последняя стабильная версия);
- **Ansible** (2.15+).

Проверка:

```bash
terraform -version
ansible --version
```

Опираться будем на заметки:

- Terraform: `[[DevOps/06_Terraform/terraform-cheatsheet]]`, `[[DevOps/06_Terraform/hcl-syntax]]`, `[[DevOps/06_Terraform/modules]]`, `[[DevOps/06_Terraform/state-management]]`, `[[DevOps/06_Terraform/best-practices]]`;
- Ansible: `[[DevOps/07_Ansible/ansible-cheatsheet]]`, `[[DevOps/07_Ansible/playbooks-and-roles]]`, `[[DevOps/07_Ansible/inventory]]`, `[[DevOps/07_Ansible/jinja2-and-vars]]`.

### Phase 5 — CI/CD

Минимум:

- аккаунт GitHub и/или GitLab;
- доступ к GitHub Actions или собственному GitLab;
- Docker Registry (может быть Docker Hub).

Для локального теста:

- Docker уже установлен на Phase 2;
- создайте тестовый репозиторий и включите:
  - GitHub Actions (`[[DevOps/05_CI-CD/github-actions]]`);
  - или GitLab CI (`[[DevOps/05_CI-CD/gitlab-ci]]`).

### Phase 6 — Monitoring & Logging

Нужно уметь:

- запускать Prometheus, Grafana, Alertmanager, Loki/ELK через Docker Compose;
- выполнять HTTP‑запросы к метрикам и логам.

Базовые знания берём из:

- `[[DevOps/08_Monitoring/prometheus]]`
- `[[DevOps/08_Monitoring/grafana]]`
- `[[DevOps/08_Monitoring/alertmanager]]`
- `[[DevOps/08_Monitoring/loki-and-logging]]`
- `[[DevOps/08_Monitoring/elk-stack]]`

### Phase 7 — Security

Нужны дополнительные утилиты:

- Trivy / Grype для сканирования образов;
- инструменты для SAST/Dependency scanning (Semgrep, `pip-audit`, `npm audit` и т.п.).

Многое будет запускаться прямо в контейнерах/CI, поэтому достаточно Docker и доступа к интернету.

---

## Облако: локалка vs реальный провайдер

Для большинства заданий достаточно:

- локального Kubernetes (minikube/kind);
- Docker Compose‑стендов.

Однако, для Terraform и финального проекта **желательно иметь аккаунт хотя бы в одном облаке**:

- AWS (`[[DevOps/09_Cloud/aws-cheatsheet]]`, `[[DevOps/09_Cloud/cloud-networking]]`);
- или GCP (`[[DevOps/09_Cloud/gcp-cheatsheet]]`);
- или Azure (`[[DevOps/09_Cloud/azure-cheatsheet]]`).

> [!warning] Важно про расходы
> - Всегда используйте **минимальные инстансы** и бесплатные квоты.  
> - Всегда удаляйте ресурсы после практики (Terraform `destroy`, удаление кластеров/VM).  
> - Отдельно следите за дисками, LB и IP‑адресами — они часто продолжают тарифицироваться после выключения VM.

---

## Что нужно знать до старта (чек‑лист)

- **Linux**:
  - Умеете открыть терминал и выполнить пару команд (`ls`, `cd`, `pwd`).
  - Понимаете, что такое пакетный менеджер (`apt`, `dnf`, `yum`).

- **Сеть**:
  - Знаете, что такое IP‑адрес и порт (детальнее разберём в `Phase_1/Module_02_Networking` и `[[DevOps/11_Networking/protocols]]`).

- **Git**:
  - Можете клонировать репозиторий и сделать `git status`.

Если чего‑то из этого нет — ничего страшного, просто заложите +1–2 недели к общей длительности и внимательно пройдите первые модули, опираясь на шпаргалки:

- Linux: `[[DevOps/01_Linux/linux-cheatsheet]]`, `[[DevOps/01_Linux/filesystem-and-permissions]]`, `[[DevOps/01_Linux/processes-and-services]]`, `[[DevOps/01_Linux/shell-scripting]]`;
- Сеть: `[[DevOps/01_Linux/networking]]`, `[[DevOps/11_Networking/troubleshooting]]`;
- Git: `[[DevOps/02_Git/git-cheatsheet]]`.

---

## Как проверять, что всё готово

Перед переходом к Phase 1 убедитесь, что:

- **Терминал и SSH работают**:

```bash
ssh localhost || echo "SSH пока не настроен — не критично, продолжайте"
```

- **Git настроен**:

```bash
git config --get user.name
git config --get user.email
```

- **Docker (для будущих фаз)**:

```bash
docker run --rm hello-world
```

Если эти команды выполняются — можно начинать курс.

