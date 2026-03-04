tags: [devops, networking-troubleshooting]

## Network Troubleshooting — traceroute, tcpdump, netstat/ss

> ⚡ Tip: Схема: DNS → ping → traceroute → ss/netstat → tcpdump. Всегда проверяй, «кто с кем и по какому порту».

### Базовый чек‑лист

1. Есть ли DNS‑резолв?
2. Доступен ли IP (ping)?
3. Куда идёт маршрут (traceroute/mtr)?
4. Кто слушает порт на сервере (ss/lsof)?
5. Доходит ли трафик (tcpdump)?

### DNS

```bash
dig example.com
dig example.com @8.8.8.8
dig +trace example.com
```

Если хост не резолвится — см. `/etc/resolv.conf`, локальные DNS‑сервисы.

### Проверка доступности

```bash
ping -c 4 8.8.8.8
ping -c 4 example.com

traceroute example.com
mtr -rw example.com
```

### Кто слушает порт (сервер)

```bash
ss -tulpn | grep 8080
sudo lsof -i :8080
```

Проверка firewall: `iptables`/`nftables` (`[[01_Linux/networking]]`), cloud SG/NSG (`[[09_Cloud/cloud-networking]]`).

### tcpdump

```bash
sudo tcpdump -i any port 80
sudo tcpdump -i eth0 host 10.0.0.10
sudo tcpdump -i eth0 dst port 443

sudo tcpdump -i eth0 -w capture.pcap
```

Анализ в Wireshark.

### Частые паттерны

- **Только некоторые клиенты не могут подключиться**:
  - проблемы маршрутизации/NAT/MTU;
  - ACL/WAF по IP.
- **Внутри кластера ок, снаружи нет**:
  - LB/Ingress/WAF, firewall.
- **Снаружи ок, внутри нет**:
  - outbound firewall, egress‑ограничения, proxy.

Связанные: `[[01_Linux/networking]]`, `[[04_Kubernetes/troubleshooting]]`, `[[09_Cloud/cloud-networking]]`, `[[10_Security/network-security]]`.

## Gotchas

- **Диагностика только с одного конца соединения**  
  - **Проблема**: не видно, кто дропает пакеты (клиент/сервер/сеть).  
  - **Решение**: собирать данные с обеих сторон (tcpdump, логи, traceroute).

- **Игнорирование MTU/PMTU проблем**  
  - **Проблема**: «подвисшие» запросы, особенно через VPN/туннели.  
  - **Решение**: проверять MTU, пробовать `ping -M do -s <size>`, настраивать MSS в туннелях/LB.

- **Слепое отключение firewall для «проверки»**  
  - **Проблема**: временная починка, но безопасность ломается.  
  - **Решение**: использовать временные и точечные правила, логировать и документировать изменения.

> ⚠️ Warning: Массовые изменения сетевых политик (iptables/nftables, cloud firewall, NetworkPolicy, WAF) в прод‑окружении без тестов и поэтапного внедрения — частая причина крупных outage. Всегда имей план отката и наблюдаемость (`[[08_Monitoring/prometheus]]`, `[[08_Monitoring/loki-and-logging]]`).

