tags: [devops, linux-networking]

## Сеть: ip, ss, iptables, tcpdump, curl, dig

### Базовая информация о сети

```bash
ip a                                  # интерфейсы и IP-адреса
ip -brief address                     # краткий вывод адресов
ip r                                  # таблица маршрутизации
ip -brief link                        # статус интерфейсов
hostname -I                           # IP-адреса хоста
cat /etc/resolv.conf                  # настройки DNS
```

### Управление интерфейсами (ip)

```bash
ip link set eth0 up                   # поднять интерфейс
ip link set eth0 down                 # опустить интерфейс

ip addr add 10.0.0.10/24 dev eth0     # добавить адрес
ip addr del 10.0.0.10/24 dev eth0     # удалить адрес

ip route add 10.10.0.0/16 via 10.0.0.1    # добавить маршрут
ip route del 10.10.0.0/16 via 10.0.0.1    # удалить маршрут
```

> ⚠️ Warning: Любые изменения маршрутов и адресов на прод‑сервере могут выбить вам SSH. Держите резервную сессию и тестируйте через `ping`/`traceroute` после изменений.

### ss / netstat — открытые порты и соединения

```bash
ss -tulpn                             # слушающие TCP/UDP сокеты с PID
ss -tn state established '( sport = :443 )'   # активные соединения с локального 443
ss -tn dst 1.1.1.1                    # соединения до 1.1.1.1

ss -s                                 # сводка по сокетам
ss -plant | grep 8080                 # кто слушает порт 8080
```

### Проверка связности: ping, traceroute, mtr

```bash
ping -c 4 8.8.8.8                     # 4 пакета до 8.8.8.8
ping -c 4 google.com                  # проверка DNS+ICMP

traceroute google.com                 # маршрут до хоста
mtr -rw google.com                    # интерактивный ping+traceroute (если установлен)
```

### DNS: dig / nslookup

```bash
dig example.com                       # A-записи
dig +short example.com                # только IP
dig +trace example.com                # пройти цепочку DNS
dig MX example.com                    # MX-записи
dig NS example.com                    # NS-записи
dig @8.8.8.8 example.com              # запрос к конкретному DNS

nslookup example.com                  # альтернативная утилита
```

### curl — HTTP(S)/TCP клиент

```bash
curl https://example.com                          # простой GET
curl -I https://example.com                       # только заголовки
curl -v https://example.com                       # подробный вывод (handshake, заголовки)
curl -k https://self-signed.local                 # игнорировать ошибки сертификата
curl -L https://example.com                       # следовать редиректам
curl -o file.html https://example.com             # скачать в файл

curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"test"}' https://api.example.com/endpoint

curl -H "Authorization: Bearer $TOKEN" https://api.example.com/me

curl --resolve example.com:443:1.2.3.4 https://example.com    # принудительно резолвить на IP
curl --http2 -v https://example.com                           # отладка HTTP/2
```

> ⚡ Tip: Для быстрой проверки API и TLS — `curl -vk https://host` и `openssl s_client -connect host:443 -servername host`.

### iptables / nftables: базовые правила

Проверка текущих правил:

```bash
iptables -L -n -v                    # фильтрующие правила (filter)
iptables -t nat -L -n -v             # таблица NAT
iptables-save | grep 80              # искать правила по порту
```

Примеры iptables:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT          # разрешить SSH
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -P INPUT DROP                                 # политика по умолчанию DROP

iptables -A INPUT -s 1.2.3.4 -j DROP                   # заблокировать IP

iptables -t nat -A PREROUTING -p tcp --dport 80 \
  -j REDIRECT --to-port 8080                           # перенаправить 80→8080
```

> ⚠️ Warning: Изменение политики по умолчанию (`iptables -P INPUT DROP`) без корректных правил для SSH может мгновенно отрезать доступ. Всегда тестируйте новые правила через отдельную SSH‑сессию.

### tcpdump — захват трафика

```bash
tcpdump -i any port 80                        # HTTP-трафик
tcpdump -i eth0 host 10.0.0.10                # трафик к/от хоста
tcpdump -i eth0 src 10.0.0.10                 # трафик от источника
tcpdump -i eth0 dst port 443                  # трафик к порту 443
tcpdump -i eth0 -nn -vvv                      # подробный вывод без резолва имён

tcpdump -i eth0 -w capture.pcap               # записать в файл (Wireshark)
tcpdump -r capture.pcap                       # читать из файла
```

Фильтры BPF:

```bash
tcpdump 'tcp[13] & 2 != 0'                    # только TCP SYN
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
                                             # только пакеты с данными (без ACK-only)
```

### Маршрутизация и таблицы

```bash
ip route show                               # текущие маршруты
ip route get 1.1.1.1                        # какой маршрут используется

ip rule show                                # правила policy routing
ip route show table 100                     # пользовательская таблица
```

### Быстрая диагностика сетевых проблем

1. **Проверить локальный хост**  
   ```bash
   ip a
   ip r
   ping -c 2 127.0.0.1
   ```
2. **Проверить шлюз и внешний мир**  
   ```bash
   ping -c 2 <gateway_ip>
   ping -c 2 8.8.8.8
   ```
3. **Проверить DNS**  
   ```bash
   dig example.com
   dig @8.8.8.8 example.com
   ```
4. **Проверить приложение**  
   ```bash
   ss -tulpn | grep 8080
   curl -v http://127.0.0.1:8080/health
   curl -v http://service.internal:8080/health
   ```

Смотрите также `[[11_Networking/troubleshooting]]` и `[[11_Networking/protocols]]` для более глубокой диагностики и протоколов.

## Gotchas

- **iptables без сохранения правил**  
  - **Проблема**: после перезагрузки все ручные изменения пропадают.  
  - **Решение**: использовать `iptables-save`/`iptables-restore` или systemd‑юниты/конфиги дистрибутива.

- **tcpdump на проде без фильтра**  
  - **Проблема**: огромный объём трафика, нагрузка на CPU/диск, возможное влияние на производительность.  
  - **Решение**: всегда задавайте интерфейс и чёткий фильтр (порт/хост/протокол).

- **curl без `-k` к self‑signed сервисам**  
  - **Проблема**: SSL ошибки маскируют реальные проблемы приложения.  
  - **Решение**: для диагностики TLS использовать `curl -vk`, а для боевого трафика правильно настроить CA/сертификаты.

- **policy routing и неожиданные маршруты**  
  - **Проблема**: `ip route` выглядит нормально, но трафик идёт по другим правилам.  
  - **Решение**: проверять `ip rule show` и отдельные таблицы, использовать `ip route get <dst>`.

> ⚠️ Warning: Работа с iptables/nftables и маршрутизацией на прод‑хостах без чёткого плана отката и out‑of‑band доступа (консоль, IPMI, облачный serial console) может привести к полной потере управления сервером.

