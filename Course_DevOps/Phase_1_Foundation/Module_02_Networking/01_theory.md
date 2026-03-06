## Module 02 — Linux Networking Basics

### Почему это важно

DevOps‑инженер половину времени диагностирует проблемы «ничего не открывается»: DNS, маршруты, firewall, порты, прокси. Умение читать сетевую конфигурацию хоста и быстро проверять «кто с кем разговаривает и по какому порту» — базовый навык.

Мы опираемся на:

- `[[DevOps/01_Linux/networking]]`
- `[[DevOps/11_Networking/protocols]]`
- `[[DevOps/11_Networking/troubleshooting]]`
- облачные сети: `[[DevOps/09_Cloud/cloud-networking]]`

---

## 1. Модель: IP → Маршрутизация → TCP/UDP → HTTP/gRPC

При диагностике сети думайте слоями:

1. **IP‑уровень** — есть ли вообще связь между хостами? (адресация, маршруты, MTU).
2. **TCP/UDP** — открыт ли нужный порт? устанавливается ли соединение?
3. **TLS** — корректен ли сертификат, нет ли проблем рукопожатия?
4. **Протокол приложения** — HTTP, gRPC, DNS, DB‑протокол и т.п.

Частая ошибка — сразу смотреть в код приложения, игнорируя первые уровни. Чек‑лист из `[[DevOps/11_Networking/troubleshooting]]` помогает не забыть базовые шаги.

---

## 2. Адресация и интерфейсы

Основные команды:

```bash
ip a              # интерфейсы и IP
ip -brief address # краткий вывод
hostname -I       # IP‑адреса хоста
cat /etc/resolv.conf
```

Что важно уметь:

- отличать `lo` (loopback), физические/виртуальные интерфейсы;
- понимать разницу между приватными и публичными адресами (10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12).

См. `[[DevOps/01_Linux/networking]]` и `[[DevOps/09_Cloud/cloud-networking]]` для контекста VPC/VNet.

---

## 3. Маршрутизация

Таблица маршрутов:

```bash
ip r
ip route get 8.8.8.8
```

Ключевые понятия:

- default‑route (`default via ...`) — куда идёт трафик «в интернет»;
- локальные сети (например, 10.0.0.0/16);
- policy routing (`ip rule show`) в более сложных схемах.

> [!warning] Ручное изменение маршрутов
> Любые `ip route add/del` на прод‑хосте могут выбить вам SSH‑доступ. Держите резервную сессию и задавайте чёткий план отката. См. предупреждения в `[[DevOps/01_Linux/networking]]`.

---

## 4. DNS

DNS отвечает за преобразование имён в IP. Примеры из `[[DevOps/01_Linux/networking]]` и `[[DevOps/11_Networking/protocols]]`:

```bash
dig example.com
dig +short example.com
dig @8.8.8.8 example.com
dig +trace example.com
```

Когда «сайт не открывается», проверяйте:

1. Резолвится ли имя?
2. Совпадает ли IP с ожидаемым?
3. Не закэширован ли старый IP (TTL, локальные кэши)?

---

## 5. Проверка связности: ping, traceroute, mtr

Проверяем доступность IP:

```bash
ping -c 4 8.8.8.8
ping -c 4 example.com
```

Путь до хоста:

```bash
traceroute example.com
mtr -rw example.com
```

Если `ping` по IP работает, а по домену — нет → проблема с DNS. Если не работает ни по IP, ни по домену → маршрутизация/firewall.

Подробнее — `[[DevOps/11_Networking/troubleshooting]]`.

---

## 6. Порты и соединения: ss / lsof

Кто слушает порты:

```bash
ss -tulpn
ss -tulpn | grep 8080
sudo lsof -i :5432
```

Полезно, когда:

- нужно выяснить, запущен ли сервис и на каком порту;
- проверить, не занял ли порт другой процесс.

---

## 7. curl как универсальный сетевой инструмент

`curl` — ваш основной «нож Swiss Army» для HTTP(S) и TCP.

Из `[[DevOps/01_Linux/networking]]`:

```bash
curl https://example.com
curl -I https://example.com           # только заголовки
curl -v https://example.com           # подробный вывод (TLS, заголовки)
curl -L https://example.com           # следовать редиректам

curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"test"}' https://api.example.com/endpoint
```

Полезные флаги:

- `-k` — игнорировать ошибки сертификата (только для диагностики);
- `--resolve` — принудительный резолвинг имени в IP;
- `--http2` — отладка HTTP/2.

---

## 8. Firewall: iptables / nftables на хосте

На Linux‑хосте чаще всего используется iptables/nftables (см. `[[DevOps/01_Linux/networking]]`):

```bash
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v
```

Пример правила:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -P INPUT DROP
```

> [!warning] Политика DROP без SSH‑правил
> Команда `iptables -P INPUT DROP` без заранее разрешённого SSH‑трафика (`--dport 22`) мгновенно отрежет вас от сервера. Всегда тестируйте изменения через отдельную SSH‑сессию и имейте out‑of‑band доступ.

В облаках дополнительно существуют security groups/NSG/WAF (`[[DevOps/09_Cloud/cloud-networking]]`, `[[DevOps/10_Security/network-security]]`).

---

## 9. tcpdump — взгляд на реальные пакеты

`tcpdump` позволяет увидеть фактический трафик:

```bash
sudo tcpdump -i any port 80
sudo tcpdump -i eth0 host 10.0.0.10
sudo tcpdump -i eth0 dst port 443
sudo tcpdump -i eth0 -w capture.pcap
```

Дальше `capture.pcap` можно открыть в Wireshark.

> [!warning] tcpdump на проде
> Не запускайте `tcpdump` без фильтра на загруженных нодах: можно забить диск и съесть CPU. Используйте конкретный интерфейс и чёткий фильтр (порт/host/proto).

Подробнее — `[[DevOps/01_Linux/networking]]`, `[[DevOps/11_Networking/troubleshooting]]`.

---

## 10. Роли Net‑инструментов в ежедневной работе DevOps

- `ip a`, `ip r` — понять, где вы и как трафик ходит.
- `ss -tulpn` — проверить, слушает ли приложение нужный порт.
- `curl -v` — проверить HTTP‑endpoint, заголовки, TLS.
- `dig` — проверить DNS.
- `tcpdump` — крайняя мера, если всё остальное непонятно.

Эти же навыки будут критичны при работе с:

- Docker‑сетями и томами (`[[DevOps/03_Docker/networking-and-volumes]]`);
- Kubernetes‑сервисами, Ingress и NetworkPolicy (`[[DevOps/04_Kubernetes/networking]]`);
- балансировщиками и WAF (`[[DevOps/11_Networking/load-balancing]]`, `[[DevOps/10_Security/network-security]]`).

---

## Что Middle DevOps должен знать по сети (чек‑лист)

- **Базовый Linux‑нетворкинг**
  - [ ] Умею читать вывод `ip a` и `ip r` и понимать, как трафик пойдёт до указанного хоста.
  - [ ] Знаю, как проверить, кто слушает порт, через `ss`/`lsof`.

- **DNS и диагностика**
  - [ ] Использую `dig`/`nslookup` для диагностики DNS‑проблем.
  - [ ] Понимаю разницу между проблемой DNS и проблемой маршрутизации.

- **HTTP/TLS диагностика**
  - [ ] Умею с помощью `curl -v` диагностировать проблемы TLS/HTTP (редиректы, коды ошибок).

- **Firewall и security**
  - [ ] Понимаю основы iptables/nftables и умею безопасно просматривать/править правила (на тестовых стендах).
  - [ ] Понимаю разницу между host‑firewall и облачными security groups/WAF.

- **Глубокая отладка**
  - [ ] Могу при необходимости снять дамп трафика `tcpdump` с аккуратным фильтром.
  - [ ] Знаю, что проверять в первую очередь при сетевых инцидентах (`[[DevOps/11_Networking/troubleshooting]]`).

Если большинство пунктов выполняется — вы готовы к практической части модуля и к работе с сетями Docker/Kubernetes в следующих фазах.

