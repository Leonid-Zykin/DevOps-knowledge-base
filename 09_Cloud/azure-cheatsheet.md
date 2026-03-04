tags: [devops, azure]

## Azure Cheatsheet — az CLI, AKS, ACR, RBAC

> ⚡ Tip: Как и с другими облаками, всегда указывай subscription и resource group явно, используй `az account show`/`az configure` для проверки текущего контекста.

### Базовая настройка az

```bash
az login
az account show
az account list --output table
az account set --subscription "SUBSCRIPTION_ID_OR_NAME"
```

Настройки:

```bash
az configure
az configure --defaults group=my-rg location=westeurope
```

### Resource groups

```bash
az group list --output table

az group create \
  --name my-rg \
  --location westeurope

az group delete \
  --name my-rg \
  --yes --no-wait
```

### Виртуальные машины

```bash
az vm list -d --output table

az vm create \
  --resource-group my-rg \
  --name web-1 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username devops \
  --generate-ssh-keys

az vm stop --resource-group my-rg --name web-1
az vm deallocate --resource-group my-rg --name web-1
az vm delete --resource-group my-rg --name web-1 --yes
```

### AKS (Azure Kubernetes Service)

Создать кластер:

```bash
az aks create \
  --resource-group my-rg \
  --name my-aks \
  --node-count 3 \
  --enable-managed-identity \
  --enable-addons monitoring \
  --generate-ssh-keys
```

Kubeconfig:

```bash
az aks get-credentials \
  --resource-group my-rg \
  --name my-aks

kubectl get nodes
```

### ACR (Azure Container Registry)

Создать ACR:

```bash
az acr create \
  --resource-group my-rg \
  --name myregistry123 \
  --sku Basic
```

Логин и push:

```bash
az acr login --name myregistry123

docker tag app:1.0 myregistry123.azurecr.io/app:1.0
docker push myregistry123.azurecr.io/app:1.0
```

### RBAC и Service Principals

Список пользователей/ролей:

```bash
az ad user list --output table
az role assignment list --assignee <object-id-or-upn> --output table
```

Создать service principal:

```bash
az ad sp create-for-rbac \
  --name "sp-deployer" \
  --role "Contributor" \
  --scopes "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/my-rg"
```

Выдаст JSON с `appId`, `password`, `tenant`.

> ⚠️ Warning: Service Principal с ролью `Owner` на subscription — почти root‑доступ. Ограничи scopes до нужных resource group/ресурсов и используй managed identities/Workload Identity, где возможно.

### Сеть и NSG

Список VNet/Subnet:

```bash
az network vnet list --output table
az network vnet subnet list \
  --resource-group my-rg \
  --vnet-name my-vnet \
  --output table
```

NSG правила:

```bash
az network nsg create \
  --resource-group my-rg \
  --name web-nsg

az network nsg rule create \
  --resource-group my-rg \
  --nsg-name web-nsg \
  --name allow-ssh \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 22 \
  --source-address-prefixes 1.2.3.4/32
```

Связанные: `[[cloud-networking]]`, `[[04_Kubernetes/kubectl-cheatsheet]]`, `[[10_Security/network-security]]`.

## Gotchas

- **Неправильный subscription/resource group**  
  - **Проблема**: ресурсы создаются и биллятся в «чужой» подписке или группе.  
  - **Решение**: проверять `az account show`, использовать `--subscription` и `--resource-group` явно.

- **Широкие роли на уровне subscription**  
  - **Проблема**: пользователи/приложения получают доступ ко всем ресурсам подписки.  
  - **Решение**: назначать роли на уровне resource group/ресурсов, использовать custom roles с минимальными правами.

- **Public IP и открытые NSG**  
  - **Проблема**: `Any` источники на SSH/RDP/HTTP без доп.защиты.  
  - **Решение**: ограничивать IP, использовать Azure Firewall/VPN/Bastion и NSG с явными правилами.

> ⚠️ Warning: Как и в других облаках, ручные изменения в Azure ресурсах поверх IaC (Bicep/Terraform) ведут к дрейфу и неожиданным результатам при последующих деплоях. Для прод‑ресурсов старайтесь, чтобы все изменения проходили через код (`[[06_Terraform/terraform-cheatsheet]]`).

