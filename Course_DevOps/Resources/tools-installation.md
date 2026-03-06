# Пошаговая установка инструментов

Подробные инструкции по установке инструментов курса. Ориентир — Ubuntu 22.04; для других ОС см. официальную документацию.

---

## Docker

### Ubuntu / Debian

```bash
# Удаление старых версий (если были)
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true

# Зависимости
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# GPG ключ Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Репозиторий
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Пользователь в группу docker
sudo usermod -aG docker "$USER"
newgrp docker

# Проверка
docker run --rm hello-world
docker compose version
```

### macOS

- Скачайте [Docker Desktop](https://www.docker.com/products/docker-desktop/) и установите.
- Или: `brew install docker docker-compose`.

### Windows

- [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/) (WSL2 backend).
- Или WSL2 + Ubuntu + установка как на Linux.

---

## kubectl

```bash
# Скачать последнюю стабильную версию
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

Для других ОС: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

---

## minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

# Запуск (требуется Docker или другой driver)
minikube start --cpus=4 --memory=8192 --driver=docker
kubectl get nodes
```

---

## kind (Kubernetes in Docker)

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Создание кластера
kind create cluster --name devops
kubectl cluster-info --context kind-devops
```

---

## Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

## Terraform

```bash
# Добавить HashiCorp repo
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
terraform -version
```

---

## Ansible

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
```

Или через pip: `pip install ansible`.

---

## Trivy

```bash
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install -y trivy
trivy --version
```

---

## SOPS (опционально)

```bash
# Через Go
go install go.mozilla.org/sops/v3/cmd/sops@latest

# Или скачать бинарь с GitHub Releases
# https://github.com/getsops/sops/releases
```

---

## Полезные утилиты (Phase 1)

```bash
sudo apt update
sudo apt install -y git curl wget htop jq net-tools traceroute tcpdump tmux
```

---

См. `[[00_PREREQUISITES]]` для обзора по фазам.
