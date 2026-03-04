tags: [devops, network-security]

## Network Security — TLS, mTLS, firewall, WAF

> ⚡ Tip: Минимальный набор: TLS везде, чёткие firewall‑правила, сегментация сетей, принцип наименьших привилегий.

### TLS / HTTPS базово

Проверка сертификата:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -dates -subject -issuer
```

Срок действия:

```bash
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -enddate
```

### mTLS (mutual TLS) концепция

- клиент и сервер проверяют сертификаты друг друга;
- обычно реализуется на уровне ingress/load balancer/sidecar (Istio/Linkerd) (`[[11_Networking/service-mesh]]`);
- требует собственного CA или интеграции с внешним PKI.

### Firewall / Security groups (пример AWS)

```bash
aws ec2 describe-security-groups \
  --query "SecurityGroups[].{ID:GroupId,Name:GroupName}" \
  --output table

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxx \
  --protocol tcp \
  --port 443 \
  --cidr 1.2.3.4/32
```

Linux‑firewall: см. `[[01_Linux/networking]]`.

### WAF (Web Application Firewall)

- Фильтрация HTTP(S) запросов:
  - OWASP rule sets;
  - rate limiting;
  - IP allow/deny lists;
  - защита от SQLi/XSS/ботов.

Примеры:

- AWS WAF + CloudFront/ALB;
- Cloud Armor (GCP);
- Azure WAF.

Связанные: `[[11_Networking/load-balancing]]`, `[[09_Cloud/cloud-networking]]`, `[[10_Security/devsecops]]`.

## Gotchas

- **Слепая вера в WAF как в «серебряную пулю»**  
  - **Проблема**: приложение остаётся уязвимым, WAF только частично фильтрует атаки.  
  - **Решение**: рассматривать WAF как дополнительный слой защиты, а не замену secure‑coding и тестам.

- **Широкие firewall‑правила ради «удобства»**  
  - **Проблема**: `0.0.0.0/0` на админ‑панели или БД.  
  - **Решение**: ограничивать доступ по IP/VPN, использовать jump‑host/bastion, admin‑панели за VPN.

- **Отсутствие мониторинга TLS‑сертификатов**  
  - **Проблема**: неожиданный expiry и простой сервиса.  
  - **Решение**: автообновление (ACME/Let’s Encrypt, cert‑manager), алерты по датам истечения.

> ⚠️ Warning: Открытые админские интерфейсы (Kibana, Prometheus, Jenkins, Argo CD, Kubernetes API) в интернет без аутентификации и WAF/Firewall — одна из самых частых причин взломов. Всегда ограничивайте доступ к подобным интерфейсам VPN/SSO и строгими сетевыми правилами.

