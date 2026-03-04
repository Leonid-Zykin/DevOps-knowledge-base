tags: [devops, gcp]

## GCP Cheatsheet — gcloud, GKE, Cloud Run, IAM

> ⚡ Tip: Всегда указывай `--project` и `--region/--zone` явно или через `gcloud config set`, чтобы не попасть в чужой проект.

### Базовая настройка gcloud

```bash
gcloud init
gcloud auth login
gcloud auth application-default login        # ADC для SDK/библиотек

gcloud config list
gcloud config set project my-project
gcloud config set compute/region europe-west3
gcloud config set compute/zone europe-west3-a
```

Список проектов:

```bash
gcloud projects list
```

### Compute Engine (VM)

Список инстансов:

```bash
gcloud compute instances list

gcloud compute instances list \
  --format="table(name,zone,status,machineType,networkInterfaces[].networkIP:label=INTERNAL_IP,tags)"
```

Создать VM:

```bash
gcloud compute instances create web-1 \
  --zone=europe-west3-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server \
  --boot-disk-size=20GB
```

SSH/стоп/удаление:

```bash
gcloud compute ssh web-1 --zone=europe-west3-a
gcloud compute instances stop web-1 --zone=europe-west3-a
gcloud compute instances delete web-1 --zone=europe-west3-a
```

### GKE (Kubernetes)

Список кластеров:

```bash
gcloud container clusters list
```

Получить kubeconfig:

```bash
gcloud container clusters get-credentials my-gke \
  --region=europe-west3 \
  --project=my-project

kubectl get nodes
```

См. `[[04_Kubernetes/kubectl-cheatsheet]]`, `[[04_Kubernetes/core-concepts]]`.

### Cloud Run

Деплой контейнера:

```bash
gcloud run deploy my-service \
  --image=eu.gcr.io/my-project/my-image:tag \
  --platform=managed \
  --region=europe-west3 \
  --allow-unauthenticated
```

Список сервисов:

```bash
gcloud run services list --region=europe-west3
```

### IAM

Сервисные аккаунты:

```bash
gcloud iam service-accounts list

gcloud iam service-accounts create deployer \
  --display-name="CI/CD Deployer"
```

Роли:

```bash
gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:deployer@my-project.iam.gserviceaccount.com" \
  --role="roles/editor"
```

Создать ключ (JSON):

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account=deployer@my-project.iam.gserviceaccount.com
```

> ⚠️ Warning: JSON‑ключи сервисных аккаунтов — чувствительные креды. Не заливай их в Git; используй секрет‑хранилища (`[[10_Security/secrets-management]]`) и по возможности Workload Identity.

### Storage (GCS)

```bash
gsutil ls
gsutil ls gs://my-bucket/**

gsutil cp file.txt gs://my-bucket/path/file.txt
gsutil cp gs://my-bucket/path/file.txt .

gsutil rsync -r ./static gs://my-bucket/static
```

Настройка версионирования:

```bash
gsutil versioning set on gs://my-bucket
gsutil versioning get gs://my-bucket
```

Связанные: `[[cloud-networking]]`, `[[08_Monitoring/prometheus]]`, `[[10_Security/network-security]]`.

## Gotchas

- **Работа в неправильном проекте**  
  - **Проблема**: создаёшь ресурсы/кластеры не в том проекте, рост неожиданных затрат.  
  - **Решение**: проверять `gcloud config list`, использовать алиасы/скрипты с явным `--project`.

- **Сервисные аккаунты с ролями Owner/Editor**  
  - **Проблема**: избыточные привилегии, особенно для CI/CD.  
  - **Решение**: использовать минимально необходимые роли, настроить IAM Conditions и аудит.

- **Открытые Cloud Storage бакеты**  
  - **Проблема**: PublicRead бакеты → утечки данных.  
  - **Решение**: использовать uniform bucket-level access, IAM, Signed URLs, минимум public.

> ⚠️ Warning: gcloud и gsutil удобны для ручных операций, но легко привести к «snowflake»‑инфраструктуре. В проде привязывайте изменения к Terraform/Deployment Manager/Helm (`[[06_Terraform/terraform-cheatsheet]]`, `[[04_Kubernetes/helm]]`), а CLI — только для диагностики и разовых действий.

