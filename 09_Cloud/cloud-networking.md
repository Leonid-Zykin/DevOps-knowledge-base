tags: [devops, cloud-networking]

## Cloud Networking — VPC, subnets, peering, VPN, CDN

> ⚡ Tip: В любом облаке базовые концепции схожи: виртуальные сети (VPC/VNet/VPC‑Network), подсети, маршрутные таблицы, security groups/firewalls, peering и gateways.

### VPC / VNet базовые элементы

- **VPC/VNet**: виртуальная приватная сеть (CIDR, регион).
- **Subnets**: разделение сети по зонам/ролям (public/private).
- **Route Tables**: правила, куда направлять трафик (local, IGW, NAT).
- **Security Groups/NSG**: stateful firewall на уровне VM/ENI.
- **NACL/Firewall rules**: stateless фильтрация на уровне subnet или глобального FW.

Пример разбиения:

- VPC CIDR: `10.0.0.0/16`
  - public subnets: `10.0.1.0/24`, `10.0.2.0/24`
  - private app: `10.0.10.0/24`, `10.0.11.0/24`
  - private db: `10.0.20.0/24`

### Public vs Private подсети

- **Public**: маршрут по умолчанию в Internet Gateway, ресурсы имеют публичные IP.
- **Private**: нет прямого выхода в интернет, только через NAT/Firewall/VPN.

Шаблон:

- LB/Ingress → public subnet;
- app‑инстансы/Pods → private app;
- DB/кэш → private db (без выхода в интернет или только через NAT с egress‑правилами).

### Peering (между VPC/VNet)

- **VPC Peering** (AWS), **VNet Peering** (Azure), **VPC Network Peering** (GCP);
- трафик идёт по приватным каналам, не выходит в интернет;
- нет транситивности (A‑B, B‑C ≠ A‑C в большинстве случаев).

Проверки:

- маршруты (route tables);
- CIDR'ы не пересекаются;
- security groups/NSG позволяют трафик.

### VPN и Direct Connect / ExpressRoute / Cloud VPN

- **Site‑to‑Site VPN**: офис/датацентр ↔ облако по зашифрованному туннелю;
- **Direct Connect/ExpressRoute**: выделенные каналы связи в облако;
- **Cloud VPN** (GCP): аналог.

Диагностика:

- проверка состояния туннелей в консоли;
- трассировка (`traceroute`, `mtr`) до приватных адресов;
- логирование VPN‑шлюзов.

### CDN

- **CloudFront (AWS)**, **Cloud CDN (GCP)**, **Azure CDN**:
  - edge‑локации;
  - кеширование статического контента;
  - TLS‑терминация ближе к клиенту.

Типичный паттерн:

- S3/GCS/Blob Storage + CDN + WAF;
- origin защищён, открыт только для CDN‑IP.

Связанные: `[[11_Networking/protocols]]`, `[[11_Networking/load-balancing]]`, `[[10_Security/network-security]]`, `[[09_Cloud/aws-cheatsheet]]`, `[[09_Cloud/gcp-cheatsheet]]`, `[[09_Cloud/azure-cheatsheet]]`.

## Gotchas

- **Пересечение CIDR между VPC и on‑prem**  
  - **Проблема**: маршруты конфликтуют, часть сетей недоступна.  
  - **Решение**: заранее планировать адресацию, избегать пересечений, использовать отдельные диапазоны для облаков.

- **Слишком «дырявые» security groups/NSG**  
  - **Проблема**: `0.0.0.0/0` для SSH/RDP/DB → открытые сервисы.  
  - **Решение**: использовать jump‑hosts/Bastion, VPN, ограниченные CIDR и WAF.

- **Зависимость прод‑трафика от одного VPN‑туннеля**  
  - **Проблема**: падение VPN = недоступность критичных сервисов.  
  - **Решение**: резервные туннели, Direct Connect/ExpressRoute, отказоустойчивые схемы маршрутизации.

> ⚠️ Warning: Изменение маршрутов, peering'ов и firewall‑правил в облаке без тестовой среды и плана отката может привести к массовым outage (недоступность всего VPC/кластера). Любые крупные сетевые изменения сначала прогоняйте в staging и документируйте.

