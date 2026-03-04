tags: [devops, aws]

## AWS Cheatsheet — EC2, S3, IAM, VPC, RDS, Lambda (CLI)

> ⚡ Tip: Настрой `aws configure` c именованными профилями (`--profile`), используй `--query` + `--output table/json` для удобного вывода.

### Общие команды AWS CLI

```bash
aws configure                                       # базовый профиль
aws configure --profile prod                        # профиль prod

aws sts get-caller-identity                         # кто я
aws ec2 describe-regions --output table

export AWS_PROFILE=prod
export AWS_DEFAULT_REGION=eu-central-1
```

### EC2

Список инстансов:

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,State:State.Name,Name:Tags[?Key=='Name']|[0].Value,AZ:Placement.AvailabilityZone}" \
  --output table
```

Запуск инстанса:

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxxxxx \
  --subnet-id subnet-xxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=demo-ec2}]'
```

Стоп/старт/терминация:

```bash
aws ec2 stop-instances --instance-ids i-123456
aws ec2 start-instances --instance-ids i-123456
aws ec2 terminate-instances --instance-ids i-123456
```

### S3

```bash
aws s3 ls                                           # список бакетов
aws s3 ls s3://my-bucket/                           # содержимое бакета

aws s3 cp file.txt s3://my-bucket/path/file.txt
aws s3 cp s3://my-bucket/path/file.txt ./file.txt

aws s3 sync ./static/ s3://my-bucket/static/ --delete

aws s3 rm s3://my-bucket/old/ --recursive
```

Версионирование:

```bash
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled
```

### IAM

Список пользователей и ролей:

```bash
aws iam list-users --output table
aws iam list-roles --output table
```

Создание пользователя:

```bash
aws iam create-user --user-name deploy
aws iam attach-user-policy \
  --user-name deploy \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

Создание access‑ключей:

```bash
aws iam create-access-key --user-name deploy
```

> ⚠️ Warning: Никогда не выдавай `AdministratorAccess` людям/приложениям без крайней необходимости. Предпочитай роли IAM с минимальными правами и временные креды через STS/SSO.

### VPC и сеть

Список VPC:

```bash
aws ec2 describe-vpcs \
  --query "Vpcs[].{ID:VpcId,CIDR:CidrBlock,IsDefault:IsDefault,Name:Tags[?Key=='Name']|[0].Value}" \
  --output table
```

Подсети:

```bash
aws ec2 describe-subnets \
  --query "Subnets[].{ID:SubnetId,VPC:VpcId,CIDR:CidrBlock,AZ:AvailabilityZone,Name:Tags[?Key=='Name']|[0].Value}" \
  --output table
```

SecurityGroups:

```bash
aws ec2 describe-security-groups \
  --query "SecurityGroups[].{ID:GroupId,Name:GroupName,VPC:VpcId}" \
  --output table
```

Добавить правило:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 1.2.3.4/32
```

Сетевые ACL, RouteTables см. `[[11_Networking/cloud-networking]]`.

### RDS

Список инстансов:

```bash
aws rds describe-db-instances \
  --query "DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Class:DBInstanceClass,Status:DBInstanceStatus,Endpoint:Endpoint.Address}" \
  --output table
```

Создание snapshot:

```bash
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snap-$(date +%Y%m%d)
```

### Lambda

Список функций:

```bash
aws lambda list-functions \
  --query "Functions[].{Name:FunctionName,Runtime:Runtime,LastModified:LastModified}" \
  --output table
```

Вызов функции:

```bash
aws lambda invoke \
  --function-name my-func \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  response.json
```

Обновление кода:

```bash
aws lambda update-function-code \
  --function-name my-func \
  --zip-file fileb://function.zip
```

Связанные: `[[cloud-networking]]`, `[[10_Security/secrets-management]]`, `[[08_Monitoring/prometheus]]`.

## Gotchas

- **Работа под root‑аккаунтом AWS**  
  - **Проблема**: любые операции выполняются с максимальными правами, высокий риск ошибок.  
  - **Решение**: использовать IAM‑пользователей/роли, root только для редких административных задач.

- **Ручные правки ресурсов мимо IaC**  
  - **Проблема**: расхождение между Terraform/CloudFormation и реальным состоянием.  
  - **Решение**: все изменения делать через IaC (`[[06_Terraform/terraform-cheatsheet]]`), AWS CLI — только для диагностики и аварий.

- **Открытые S3‑бакеты и security groups**  
  - **Проблема**: `0.0.0.0/0` на 22/80/443 или PublicRead на S3 — классические векторы утечки.  
  - **Решение**: ограничивать CIDR/приватный доступ, использовать S3 Block Public Access, WAF и IAM policies (`[[10_Security/network-security]]`).

> ⚠️ Warning: Долгоживущие access‑ключи (особенно с широкими правами) хранящиеся в .env/конфигах без ротации — частая причина серьёзных инцидентов в AWS. Используйте IAM роли, STS, SSO и централизованное управление секретами (`[[10_Security/secrets-management]]`).

